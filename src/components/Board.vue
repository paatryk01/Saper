<template>
    <div id="Board">
        <!-- <button class="startButton" @click="startNewGame">New Game</button> -->
        <div class="grid">
            <div class="square"
            v-for="(square, index) in squares"
            :id="index"
            :key="index"
            :class="squares[index]"
            @click="clicked()"
            @click.right.prevent="addFlag"
            >
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Board',
    props: {
        width: Number, 
        bombAmount: Number
    },
    data() {
        return {
            squares: [],
            flags: 0,
            isGameOver: false,
            cells: [],
            classesToDelete: ['one', 'two', 'three', 'four', 'bomb', 'flag', 'checked']
        }
    },
    methods: {

        //method for start new game isn't finished yet
        
        startNewGame() {
            for(let i = 0; i < this.cells.length; i++) {
                this.cells[i].classList.remove(...this.classesToDelete)
                this.cells[i].innerText = '';
                this.cells[i].removeAttribute('style'); 
                this.cells[i].removeAttribute('data');
            }

            this.isGameOver = false;
            this.flags = 0;
            this.squares = [];
            this.cells = [];
            }
            this.createBoard()
            this.fillCells()
        },

        /**createBoard creates shuffled array with 'empty' and 'bomb' classes*/

        createBoard() {

            const bombsArray = Array(this.bombAmount).fill('bomb');
            const emptyArray = Array(this.width * this.width - this.bombAmount).fill('empty');
            const gameArray = emptyArray.concat(bombsArray);
            const shuffledArray = gameArray.sort(() => Math.random() - 0.5);
            this.squares = [...shuffledArray]
        },

        /**fillCells creates an array from elements which contains class 'square' and then invoke addNumbers method */

        fillCells() {

            const elements = document.getElementsByClassName('square');
            this.cells = [...elements];

            this.addNumbers()
        },
    
        /**clicked method checks if 'square' is undefined, because method is using in two situations:
        after click and in this situation 'square' is event.target
        in recursion and in this situation 'square' is from checkSquere method

        method is checking conditions if is game over or 'square' is checkec or has a flag
        if square has a 'bomb' class it's game over
        in other situation method is getting 'data' attribute - 'total' 
        if clicked 'square' isn't empty, method is adding a total number of contacts with bomb and display it
        if clicked 'square' is empty, method invokes checkSquare method - recursion */

        clicked(square) {   

            if(square === undefined) {
                square = event.target;
            }

            let currentId = square.id;

            if(this.isGameOver) return;
            if(square.classList.contains('checked') || square.classList.contains('flag')) return
            if(square.classList.contains('bomb')) {
                this.gameOver();
            } else {
                let total = square.getAttribute('data');

                if(total != 0) {
                    square.classList.add('checked');
                    if(total == 1) square.classList.add('one');
                    if(total == 2) square.classList.add('two');
                    if(total == 3) square.classList.add('three');
                    if(total == 4) square.classList.add('four');
                    square.innerHTML = total;
                    return
                }
                this.checkSquare(currentId);
            }
            square.classList.add('checked');
        },

        /**method gameOver checks every cell and show bomb icon if cell contains 'bomb' class */

        gameOver() {   

            this.isGameOver = true;

            this.cells.forEach((square) => {
                if(square.classList.contains('bomb')) {
                    square.innerHTML = '💣';
                    square.style = "background-color: #ff0000"
                }
            })
            this.$emit('gameOverState', this.isGameOver);
            console.log('Game over');
        },

        /**method is adding flag and checks if clicked cell has a flag and if is able to add flag (limit is equal to bomb amount)
        addFlag calls checkForWin each time the flag is added */

        addFlag() {

            let square = event.target;
            
            if(this.isGameOver) return
            if(!square.classList.contains('checked') && (this.flags < this.bombAmount)) {
                if(!square.classList.contains('flag')) {
                    square.classList.add('flag');
                    square.innerHTML = '🚩';
                    this.flags++;
                    this.checkForWin();
                } else {
                    square.classList.remove('flag');
                    square.innerHTML = '';
                    this.flags--;
                }
            }
            this.$emit('flags', this.flags)
        },

        /**method checks contacts with bomb for each 'square' and set attribute 'data' with number of contacts 
        method checks contacts in every direction */

        addNumbers() {

            for(let i = 0; i < this.cells.length; i++) {
                let total = 0;
                const isLeftEdge = (i % this.width === 0);
                const isRightEdge = (i % this.width === this.width - 1);
                
                if(this.cells[i].classList.contains('empty')) {
                    if(i > 0 && !isLeftEdge && this.cells[i - 1].classList.contains('bomb')) total++;
                    if(i > 9 && !isRightEdge && this.cells[i + 1 - this.width].classList.contains('bomb')) total++;
                    if(i > 9 && this.cells[i - this.width].classList.contains('bomb')) total++;
                    if(i > 11 && !isLeftEdge && this.cells[i - 1 - this.width].classList.contains('bomb')) total++;
                    if(i < 99 && !isRightEdge && this.cells[i + 1].classList.contains('bomb')) total++;
                    if(i < 90 && !isLeftEdge && this.cells[i - 1 + this.width].classList.contains('bomb')) total++;
                    if(i < 88 && !isRightEdge && this.cells[i + 1 + this.width].classList.contains('bomb')) total++;
                    if(i < 90 && this.cells[i + this.width].classList.contains('bomb')) total++

                    this.cells[i].setAttribute('data', total);
                }
            }
        },

        /**recursion is checking every cell around clicked 'square' to find empty cells and using clicked method  */

        checkSquare(currentId) {

            const isLeftEdge = (currentId % this.width === 0);
            const isRightEdge = (currentId % this.width === this.width - 1);

            setTimeout(() => {
                
                if(currentId > 0 && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare)
                }

                if(currentId > 9 && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1 - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId > 9) {
                    const newId = this.cells[parseInt(currentId) - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId > 11 && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1 - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId < 99 && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId < 90 && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1 + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId < 88 && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1 + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }

                if(currentId < 90) {
                    const newId = this.cells[parseInt(currentId) + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
            }, 10)
        },

        /**method is checking if 'square' with 'bomb' has a flag
        if this 2 numbers are equal player won */

        checkForWin() {

            let matches = 0;

            for(let i = 0; i < this.cells.length; i++) {
                if(this.cells[i].classList.contains('flag') && this.cells[i].classList.contains('bomb')) {
                    matches++;
                }
            }

            if(matches === this.bombAmount) {
                alert('congratulations! you are a winner');
                this.isGameOver = true;
            }
        }
    },
    created() {
        this.createBoard()
    },
    mounted() { 
        this.fillCells()    
    }
}
</script>


<style>
    .grid {
        height: 400px;
        width: 400px;
        display: flex;
        flex-wrap: wrap;
        background-color:rgb(235, 230, 223);
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 240px;
    }

    .square {
        height: 40px;
        width: 40px;
        box-sizing: border-box;
        border: 2px solid;
        border-color: #eee #999 #999 #eee;
        background-color: #ccc;
        font-weight: 600;
    }

    .bomb {
        /* background-color: red; */
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .checked {
        color: #0000ff;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #bbb;
        border-width: 1px;
        border-color: #999;
        padding: 1px;
    }

    .flag {
        display:flex;
        justify-content: center;
        align-items: center;
    }

    .startButton {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        background-color: #ffff00;
        color: black;
        border: 1px solid #000;
        padding: 6px 12px;
    }

    .startButton:hover {
        cursor: pointer;
        transition: 1s;
        background-color: #000;
        color: #fff;
    }

    .one {
        color: #0000ff;
    }

    .two {
        color: #008500;
    }

    .three {
        color: #ff0000;
    }

    .four {
        color: #060073;
    }
</style>
