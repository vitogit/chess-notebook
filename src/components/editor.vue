<template>
  <div>
    <div class="spare">
      <div @mousedown="dragPiece('king', 'black')" class="piece king black"></div>
      <div @mousedown="dragPiece('queen', 'black')" class="piece queen black"></div>
      <div @mousedown="dragPiece('rook', 'black')" class="piece rook black"></div>
      <div @mousedown="dragPiece('bishop', 'black')" class="piece bishop black"></div>
      <div @mousedown="dragPiece('knight', 'black')" class="piece knight black"></div>
      <div @mousedown="dragPiece('pawn', 'black')" class="piece pawn black"></div>
    </div>
    <div class="blue merida">
      <div ref="board" class="cg-board-wrap"></div>
    </div>
    <div class="spare">
      <div @mousedown="dragPiece('king', 'black')" class="piece king white"></div>
      <div @mousedown="dragPiece('queen', 'white')" class="piece queen white"></div>
      <div @mousedown="dragPiece('rook', 'white')" class="piece rook white"></div>
      <div @mousedown="dragPiece('bishop', 'white')" class="piece bishop white"></div>
      <div @mousedown="dragPiece('knight', 'white')" class="piece knight white"></div>
      <div @mousedown="dragPiece('pawn', 'white')" class="piece pawn white"></div>
    </div>
  </div>
</template>

<script>
import Chess from 'chess.js'
import {Chessground} from 'chessground'

export default {
  name: 'chessboard',
  props: {
    shapes: {
      type: Array,
      default: () => [],
    },
    fen: {
      type: String,
      default: '',
    }
  },
  data () {
    return {
      local_fen: this.fen
    }
  },
  watch: {
    fen: function (newFen) {
      this.fen = newFen
      this.local_fen = newFen
      this.loadPosition()
    },
    shapes: function (newShapes) {
      this.board.setShapes(newShapes)
    }
  },
  methods: {
    dragPiece(piece, color) {
      this.board.dragNewPiece({
        color: color,
        role: piece
      }, event, true);
    },
    sendInfo() {
      let info = []
      info['fen'] = this.local_fen
      info['shapes'] = this.board.state.drawable.shapes
      this.$emit('onMove', info)
    },
    loadPosition () {
      this.board = Chessground(this.$refs.board, {
        fen: this.local_fen,
        movable: {
          free: true,
          color: 'both',
          showDests: true
        },
        drawable: {
          enabled: true
        },
        draggable: {
          deleteOnDropOff: true
        },
        events: {
          dropNewPiece: (piece, key) => {
            // Move the existing king, even when dropping a new one.
            if (piece.role !== 'king') return;
            const king = Object.keys(this.board.state.pieces).find(key => {
              const p = this.board.state.pieces[key];
              return p.role === 'king' && p.color === piece.color;
            });
            const diff = {};
            if (king) diff[king] = null;
            diff[key] = piece;
            this.board.setPieces(diff);
          },
          change: () => {
            this.local_fen = this.board.getFen()
            this.sendInfo()
          }
        }
      })
      this.board.setShapes(this.shapes)
    },
  },
  mounted () {
    this.loadPosition()
  },
  created () {
    this.board = null
    this.promotions = []
    this.promoteTo = 'q'
  },
}
</script>
