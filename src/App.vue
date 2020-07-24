<template>
  <div id="app">
    <Header 
      :bombAmount='bombAmount'
      :flags='flags'
    />
    <Board 
      :width='width'
      :bombAmount='bombAmount'
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
      flags: 0
    }
  },
  methods: {
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
