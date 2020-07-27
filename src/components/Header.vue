<template>
    <div>
        <h1>MineSweeper</h1>
        <h3>Marked fields: {{ flags }} 🚩 / {{ bombAmount }} 💣</h3>
    </div>
</template>

<script>
    export default {
    name: 'Header',
    props: {
        bombAmount: Number,
        flags: Number
        }
    }
</script>

<style scoped>

h1 {
    font-size: 40px;
}

h1, h3{
    text-align: center;
}

</style>
