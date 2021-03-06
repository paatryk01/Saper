<template>
    <div id="Board">
        <form class="options" @submit.prevent='prepareNewGame'>
            <p>Size</p>
            <input type="number" class="input-number" 
            :min='fieldSizeMin' 
            :max='fieldSizeMax'
            v-model.number='width'>
            <p>Bombs Amount</p>
            <input type="number" class="input-number"
            :min='bombsMin' 
            :max='bombsMax'
            v-model.number='bombsAmount'>
            <button class="startButton">New game</button>
        </form>
        <h2 class='gameState' v-text='this.gameState' />
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
        states: Object
    },
    data() {
        return {
            width:10,
            fieldSizeMin: 5,
            fieldSizeMax: 20,
            bombsMin: 5,
            bombsMax: 40,
            bombsAmount: 10,
            gameState: this.states.start,
            squares: [],
            flags: 0,
            isGameOver: false,
            cells: [],
            classesToDelete: ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'bomb', 'flag', 'checked'],
            topEdge: this.topEdge,
            topLeftCorner: this.topLeftCorner,
            lastCell: this.lastCell,
            downEdge: this.downEdge,
            downRightCorner: this.downRightCorner    
        }
    },
    methods: {
        
        /** method to restart game */
        async prepareNewGame() {
            const grid = document.querySelector('.grid');
            if(this.bombsAmount < this.width * this.width) {
                this.$emit('bombsAmount', this.bombsAmount);
            } else {
                alert(`You can't set more bombs than fields.`)
                this.bombsAmount = 10;
            }
            
            grid.style.width = this.width * 40 + 'px'
            await this.clearHelper();
            await this.createBoard();
            await this.fillCells();
            
        },
        /** createBoard creates shuffled array with 'empty' and 'bomb' classes */
        createBoard() {
            
            const bombsArray = Array(this.bombsAmount).fill('bomb');
            const emptyArray = Array(this.width * this.width - this.bombsAmount).fill('empty');
            const gameArray = emptyArray.concat(bombsArray);
            const shuffledArray = gameArray.sort(() => Math.random() - 0.5);
            this.squares = [...shuffledArray]
            this.setEdges()
        },
        /** fillCells creates an array from elements which contains class 'square' and then invoke addNumbers method */
        fillCells() {
            const elements = document.getElementsByClassName('square');
            this.cells = [...elements];
            this.addNumbers()
        },
    
        /** clicked method checks if 'square' is undefined, because method is using in two situations:
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
            if(square.classList.contains('bomb')) this.gameOver()
            let total = square.getAttribute('data');
            const totalClasses = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight'];
            if(total > 0 && total <= totalClasses.length) {
                square.classList.add('checked');
                square.classList.add(totalClasses[total-1])
                square.innerHTML = total;
                if(total > 3) {
                    this.gameState = this.states.soClose;
                }
                return
            }
            if(!square.classList.contains('bomb')) {
                this.gameState = this.states.niceTry;
            }
            
            this.checkSquare(currentId);
            square.classList.add('checked');
        },
        /** method is adding flag and checks if clicked cell has a flag and if is able to add flag (limit is equal to bomb amount)
        addFlag calls checkForWin each time the flag is added */
        addFlag() {
            let square = event.target;
            
            if(this.isGameOver) return
            if(!square.classList.contains('checked') && (this.flags <= this.bombsAmount)) {
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
        /** method checks contacts with bomb for each 'square' and set attribute 'data' with number of contacts 
        method checks contacts in every direction */
        addNumbers() {
            for(let i = 0; i < this.cells.length; i++) {
                let total = 0;
                const isLeftEdge = (i % this.width === 0);
                const isRightEdge = (i % this.width === this.width - 1); 
                
                if(this.cells[i].classList.contains('empty')) {
                    if(i > 0 && !isLeftEdge && this.cells[i - 1].classList.contains('bomb')) total++;
                    if(i > this.topEdge && !isRightEdge && this.cells[i + 1 - this.width].classList.contains('bomb')) total++;
                    if(i > this.topEdge && this.cells[i - this.width].classList.contains('bomb')) total++;
                    if(i > this.topLeftCorner && !isLeftEdge && this.cells[i - 1 - this.width].classList.contains('bomb')) total++;
                    if(i < this.lastCell && !isRightEdge && this.cells[i + 1].classList.contains('bomb')) total++;
                    if(i < this.downEdge && !isLeftEdge && this.cells[i - 1 + this.width].classList.contains('bomb')) total++;
                    if(i < this.downRightCorner && !isRightEdge && this.cells[i + 1 + this.width].classList.contains('bomb')) total++;
                    if(i < this.downEdge && this.cells[i + this.width].classList.contains('bomb')) total++
                    this.cells[i].setAttribute('data', total);
                }
            }
        },
        /** recursion is checking every cell around clicked 'square' to find empty cells and using clicked method  */
        checkSquare(currentId) {
            const isLeftEdge = (currentId % this.width === 0);
            const isRightEdge = (currentId % this.width === this.width - 1);
            setTimeout(() => {
                
                if(currentId > 0 && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare)
                }
                if(currentId > this.topEdge && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1 - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId > this.topEdge) {
                    const newId = this.cells[parseInt(currentId) - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId > this.topLeftCorner && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1 - this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId < this.lastCell && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId < this.downEdge && !isLeftEdge) {
                    const newId = this.cells[parseInt(currentId) - 1 + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId < this.downRightCorner && !isRightEdge) {
                    const newId = this.cells[parseInt(currentId) + 1 + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
                if(currentId < this.downEdge) {
                    const newId = this.cells[parseInt(currentId) + this.width].id;
                    const newSquare = document.getElementById(newId);
                    this.clicked(newSquare);
                }
            }, 10)
        },
        /** method gameOver checks every cell and show bomb icon if cell contains 'bomb' class */
        gameOver() {   
            this.isGameOver = true;
            this.gameState = this.states.lost;
            this.cells.forEach((square) => {
                if(square.classList.contains('bomb')) {
                    square.innerHTML = '💣';
                    square.style = "background-color: #ff0000"
                }
            })
        },
        /** method is checking if 'square' with 'bomb' has a flag
        if this 2 numbers are equal player won */
        checkForWin() {
            let matches = 0;
            for(let i = 0; i < this.cells.length; i++) {
                if(this.cells[i].classList.contains('flag') && this.cells[i].classList.contains('bomb')) {
                    matches++;
                }
            }
            if(matches === this.bombsAmount) {
                this.isGameOver = true;
                this.gameState = this.states.won;
                alert('Congratulations, you are a winner! 🏆')
            }
        },
        /** method to clear all board to start new game */
        clearHelper() {
            for(let i = 0; i < this.cells.length; i++) {
                this.cells[i].classList.remove(...this.classesToDelete)
                this.cells[i].innerText = '';
                this.cells[i].removeAttribute('style'); 
                this.cells[i].removeAttribute('data');
            }
            
            this.setEdges();
            this.gameState = this.states.start;
            this.squares = [];
            this.cells = [];
            this.flags = 0;
            this.isGameOver = false;
            this.$emit('flags', this.flags);   
        },
        
        setEdges() {
            this.topEdge = this.width - 1;
            this.topLeftCorner =  this.width;
            this.lastCell = (this.width * this.width) - 1;
            this.downEdge = (this.width * this.width) - this.width;
            this.downRightCorner = (this.width * this.width) - this.width - 1;
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
        width: 400px;
        display: flex;
        flex-wrap: wrap;
        background-color:rgb(235, 230, 223);
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        top: 240px;
        margin: 30px 0px 150px
    }
    .square {
        height: 40px;
        width: 40px;
        box-sizing: border-box;
        border: 2px solid;
        border-color: #eee #999 #999 #eee;
        background-color: #ccc;
        font-weight: 500;
    }
    .square:hover {
        background-color: #ddd;
    }
    .gameState {
        text-align: center;
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
        font-family: 'Righteous';
        text-transform: uppercase;
        background-color: transparent;
        color: #000;
        border: 2px solid #000;
        padding: 6px 12px;
    }
    .startButton:hover {
        cursor: pointer;
        transition: 1s;
        background-color: #000;
        color: #fff;
        border-radius: 0px 8px 0px 8px;
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
    .five, .six, .seven, .eight {
        color: #070077;
    }
    .options {
        display: flex;
        justify-content: center;
        font-weight: 600;
    }
    .options input{
        margin: 0 15px;
        background: transparent;
        border: 0px;
        border-bottom: 2px solid #000;
        font-family: 'Righteous';
        font-size: 16px;
        text-align: center;
    }
</style>