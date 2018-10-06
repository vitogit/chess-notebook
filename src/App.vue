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
          </div>
        </nav>
      </div>
      <div>
        <div class="container">
          <div class="columns">
            <div class="column">
              <chessboard :fen="fen" @onMove="onMove"/>
            </div>
            <div class="column">
              <b-field label="Title">
                  <b-input v-model="title"></b-input>
              </b-field>
              <b-field label="Fen">
                  <b-input v-model="fen"></b-input>
              </b-field>
              <b-field label="Tags">
                  <b-taginput
                      v-model="tags"
                      ellipsis
                      icon="label"
                      placeholder="Add a tag">
                  </b-taginput>
              </b-field>
              <p class="control">
                <button @click="savePosition" class="button is-primary">Save</button>
              </p>
            </div>
            <div class="column">
              <b-field label="Notes">
                  <b-input type="textarea" v-model="notes"></b-input>
              </b-field>

            </div>
          </div>


          <div class="columns is-multiline"> 
            <div  v-for="(t, index) in positions" class="column is-one-third">
              <div class="card">
                <div class="card-header">
                  <p v-html="t.title" class="card-header-title"></p>
                </div>
                <div class="card-content">
                  <chessboard :fen="t.fen" :shapes="t.shapes"/>
                  <div v-html="t.notes" class="content is-size-7"></div>
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
import './components/style/theme.css'

import Chess from 'chess.js'

export default {
  name: 'app',
  components: {
    chessboard
  },
  data () {
    return {
      username: 'e4guardian',
      positions: [],
      tags: [],
      fen: '',
      notes: '',
      title: '',
    }
  },
  methods: {
    savePosition() {
      //TODO improve this
      let shapes = vm.$children[0].$children[0].board.state.drawable.shapes
      console.log("shapes________",shapes)
      
      let position = {
        title: this.title,
        fen: this.fen,
        shapes: shapes,
        notes: this.notes,
        tags: this.tags
      }
      console.log("position________",position)
      this.positions.push(position)
    },
    onMove(data) {
      console.log("data________",data)
      this.fen = data.fen
    }

  },
  created() {
      let position = {
        title: 'Break e4',
        fen: 'rnbqkbnr/pp1ppppp/8/2p5/4P3/8/PPPP1PPP/RNBQKBNR w KQkq - 0 2',
        shapes: [{orig: "f4", pos: [242,121], brush: "green", mouseSq: "f4", dest: undefined}],
        notes: 'Break with e4 to break the center',
        tags: ['breaks', 'center']
      }
      this.positions.push(position)
  }, 
  mounted() {

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
