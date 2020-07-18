<template>
  <div id="app">
    <Header 
      :bombAmount='bombAmount'
      :isGameOver='isGameOver'
      :flags='flags'
    />
    <Board 
      :width='width'
      :bombAmount='bombAmount'
      @gameOverState='updateState'
      @flags='updateFlags'
    />
  </div>
</template>

<script>
import Header from './components/Header.vue'
import Board from './components/Board.vue'

export default {
  name: 'App',
  components: {
    Header,
    Board
  },
  data() {
    return {
      width: 10,
      bombAmount: 20,
      isGameOver: false,
      flags: 0
    }
  },
  methods: {
    updateState(state) { 
      this.isGameOver = state;
    },

    updateFlags(flags) {
      this.flags = flags;
    }
  },
  created() {

  }
}
</script>

<style>

</style>
