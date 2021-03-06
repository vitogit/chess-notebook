<template>
  <div id="app">
    <section class="hero is-fullheight">
      <div class="hero-head">
        <nav class="navbar">
        
          <div class="container">
            <div>
              <a href="/">
                <p class="title">Chess Notebook</p>
                <p class="subtitle">Save chess concepts, positions and motives.</p>
              </a>
            </div>
            <div id="navbarMenu" class="navbar-menu">
              <div class="navbar-end"></div>
            </div>
            <div class ="navbar-item">
              <span style="font-size: 12px;font-weight: normal;" v-if="logged">
                  {{useremail}} <br> 
                  <button class="button is-info is-small" @click="signOff()">Exit</button>
              </span>
              <span v-if="!logged"><button class="button is-info" @click="signIn()">Sign-in</button></span>
            </div>  
          </div>
        </nav>
      </div>
      <div>
        <div class="container">
          <!-- <iframe width=600 height=397 frameborder=0
          src="https://lichess.org/analysis?fen=rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR%20b%20KQkq%20-%200%201/embed/?theme=auto&bg=auto"
          ></iframe> -->
          <div class="columns">
            <div class="column">
              <editor ref="currentBoard" :shapes="currentPosition.shapes" :fen="currentPosition.fen" @onMove="onMove"/>
            </div>
            <div class="column">
              <b-field label="Title">
                  <b-input v-model="currentPosition.title"></b-input>
              </b-field>
              <b-field label="Fen">
                  <b-input custom-class="tiny-font" v-model="fen"></b-input>
              </b-field>
              <b-field label="Id">
                  <b-input v-model="currentPosition.id"></b-input>
              </b-field>
              <b-field label="Tags">
                  <b-taginput
                      v-model="currentPosition.tags"
                      ellipsis
                      icon="label"
                      placeholder="Add a tag">
                  </b-taginput>
              </b-field>
              <p class="control">
                <button @click="savePosition" class="button is-primary">Save</button>
                <button @click="newPosition" class="button is-primary">New</button>
                <button @click="duplicate" class="button is-primary">Duplicate</button>
              </p>
            </div>
            <div class="column">
              <b-field label="Notes">
                  <b-input type="textarea" v-model="currentPosition.notes"></b-input>
              </b-field>
              <a class="button" target="_blank" :href="'https://lichess.org/analysis?fen='+fen"> Analyze fen</a>
            </div>
          </div>
          <div>
            <b-field>
              <b-checkbox-button v-for="tag in allTags"
                  :native-value="tag"
                  v-model="selectedTags"
                  @click="reloadPositions()">
                  <span>{{tag}}</span>
              </b-checkbox-button>
            </b-field>
          </div>
          <br>
          <div class="columns is-multiline" :key="positionsComponentKey"> 
            <div v-for="(t, index) in filterByTags(positions)" class="column is-one-third" >
              <div class="card" v-show="!t.hide">
                <div class="card-header">
                  <a @click="selectPosition(t)" v-html="t.title" class="card-header-title"></a>
                  <a @click="removePosition(t)" href="#" class="card-header-icon"> x </a>
                  {{t.id}}
                </div>
                <div class="card-content">
                  <chessboard :fen="t.fen" :shapes="t.shapes"/>
                  <div v-html="t.notes" class="content is-size-6"></div>
                  Tags: <div v-html="t.tags" class="content is-size-7"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="hero-foot">
        <div class="container">
          <div class="has-text-centered">
            Made by <a href="http://vitomd.com">@vitomd</a> with <a href="https://vuejs.org/">Vue.js</a> | Check all my <a href="http://vitomd.com/blog/projects/">chess related projects</a>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import chessboard from './components/chessboard'
import editor from './components/editor'
import './components/style/theme.css'
import './components/style/editor.css'

import Chess from 'chess.js'

export default {
  name: 'app',
  components: {
    chessboard,
    editor
  },
  data () {
    return {
      positions: [],
      selectedTags: [],
      allTags: [],
      logged: false,
      fen: '',
      currentPosition: {},
      maxId: 0,
      currentFile: {},
      positionsComponentKey: 0
    }
  },
  watch: {
    fen: function (newFen) {
      this.currentPosition.fen = newFen
    },
    currentPosition: function (newPosition) {
      this.fen = newPosition.fen
    }
  },
  methods: {
    selectPosition(p) {
      this.currentPosition = { ...p } // Clone the oject
      this.fen = p.fen
    },
    removePosition(pos) {
      this.positions = this.positions.filter(p => p.id !== pos.id)
    },
    newPosition() {
      this.currentPosition = {}
      this.fen = ''
    },
    duplicate() {
      this.currentPosition.id = undefined
      this.currentPosition.title = 'Copy of ' + this.currentPosition.title
    },
    savePosition() {
      //TODO improve this
      let self = this
      let shapes = this.$refs.currentBoard.board.state.drawable.shapes
      this.currentPosition.fen = this.fen
      this.currentPosition.shapes = shapes
      if (this.currentPosition.id) {
        this.positions.forEach(function(pos,index) { 
           if (pos.id == self.currentPosition.id) {
             // Clone the oject and use splice to invoke reactivity
             self.positions.splice(index, 1, { ...self.currentPosition })
           }
        }) 
      } else {
        this.currentPosition.id = this.getMaxId(this.positions) + 1
        let tempPosition = { ...this.currentPosition }
        this.positions.unshift(tempPosition)
      }
      this.saveFile()
    },
    onMove(data) {
      this.fen = data.fen
    },
    saveFile() {
      let self = this 
      self.currentFile.name = 'chess-notebook-positions.json'
      self.currentFile.content = JSON.stringify(this.positions)
      window.driveService.saveFileRaw(self.currentFile, function(file){
        self.currentFile = file
        self.$toast.open({
            message: `${file.name} - Saved!`,
            type: 'is-success'
        })
      })
    },
    loadFile(err, files) {
      if (files.length > 0) {
        // let loading = this.$loading.open()
        let tempFile = files[0]
        let self = this
        driveService.loadFileRaw(tempFile, function(file){
          self.currentFile = file
          self.positions = JSON.parse(file.content)
          self.maxId = self.getMaxId(self.positions)
          // loading.close()
          self.$buefy.toast.open({
              message: `${file.name} - Loaded!`,
              type: 'is-success'
          })
          self.loadTags()
        })
      }

    },
    getMaxId(positions) {
      if (positions.length > 0) {
        let ids = positions.map(p => p.id || 0)
        return Math.max(...ids)
      } else {
        return 0
      }

    },
    signIn() {
      window.loginService.signIn()
    },
    signOff(){
      window.loginService.signOut()
    },
    initClient() {
      window.loginService.initClient(this.updateSigninStatus)
    },
    updateSigninStatus(isSignedIn) {
      this.logged = isSignedIn 
      if (this.logged) {
        this.useremail = window.loginService.userProfile().getEmail()
        window.driveService.listFiles('chess-notebook-positions', this.loadFile)
      }
    },
    filterByTags() {
      if (this.selectedTags.length > 0) {
        return this.positions.filter(p => p.tags.some(r => this.selectedTags.indexOf(r) >= 0))
      } else {
        return this.positions
      }
    },
    reloadPositionsComponent() {
      this.positionsComponentKey += 1
    },
    loadTags() {
      let tempTags = new Set()
      this.positions.forEach(function(pos,index) { 
        if (pos.tags) {
          pos.tags.forEach(tag => tempTags.add(tag))
        }
      }) 
      this.allTags = Array.from(tempTags)
    }
  },
  created() {
    const SCOPES = 'https://www.googleapis.com/auth/drive.file'
    const CLIENT_ID = '787907413982-ek7jje54nljmljno0rja381lg5hsan6h.apps.googleusercontent.com'
    const DISCOVERY_DOCS = ['https://www.googleapis.com/discovery/v1/apis/drive/v3/rest']
    window.loginService = new LoginService(CLIENT_ID, SCOPES, DISCOVERY_DOCS)
    window.driveService = new DriveService()
  }, 
  mounted() {
    window.gapi.load('auth2', this.initClient)
  }
}
</script>

<style>
  .hero-head {
    padding-top: 10px;
  }
  .hero-foot {
    padding-bottom: 10px;
  }
  pre, code {
      white-space: pre-line;
      padding: 10px
  }
  .small-note {
    border: 1px solid;
    padding: 10px;
    margin: 10px;
    width: 270px;
    float: left;
  }
  .container {
    width: 95%;
    max-width: 95%;
  }

</style>
