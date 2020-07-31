<template>
    <div id="app">
        <Header 
        :bombsAmount='bombsAmount'
        :flags='flags'
        />
        <Board 
        @flags='updateFlags'
        @bombsAmount='updateBombsAmount'
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
                flags: 0,
                bombsAmount: 10
            }
        },
        methods: {
            updateFlags(flags) {
                this.flags = flags;
            },
            updateBombsAmount(bombsAmount) {
                this.bombsAmount = bombsAmount;
            }

        },
        created() {

        }
    }
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Righteous&display=swap');

body {
    background: linear-gradient(160deg, #a6e0d9, #3dac9f);
    background-repeat: no-repeat;
    background-size: contain;
    height: 1200px;
    font-family: 'Righteous', cursive;
    text-transform: uppercase;
}

</style>
