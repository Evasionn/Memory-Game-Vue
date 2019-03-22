<template>
  <div class="container">
    <div class="row">
      <div class="col-md-12 text-center">
       <div class="header">
        <h1>Memory Game</h1>
       </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-8">
        <div class="game-board">
          <table>
            <tr v-for="(row, rowIndex) in board" :key="rowIndex">
              <td v-for="(col, colIndex) in row" :key="colIndex">
                <img src="../assets/0.png" v-if="col.status === 0" class="game-asset" v-on:click="showImage(rowIndex, colIndex)">
                <img :src="getImgUrl(col.image)" v-if="col.status === 1" class="game-asset">
              </td>
            </tr>
          </table>
        </div>
     </div>
     <div class="col-md-4">
     <table width="100%" class="score-board">
      <tr>
        <td class="text-center" width="40%"> <span class="bold">Left:</span> {{left}} </td>
      </tr>
      <tr>
        <td class="text-center"> <span class="bold">Number of Clicks:</span> {{move}} </td>
      </tr>
      <tr>
        <td class="text-center"> <span class="bold">Elapsed Time: </span> <span v-if="elapsedTime.minute">{{elapsedTime.minute}}mins and </span>{{elapsedTime.second}} secs </td>
      </tr>
      <tr>
        <td class="text-center">
         <button class="btn btn-primary" v-on:click="newGame()">New Game</button>
         <button class="btn btn-primary" v-on:click="showHint()"> Hint ({{hintLeft}}) </button>
       </td>
      </tr>
     </table>
     </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'GameBoard',
  mounted() {
    this.newGame();
  },
  methods: {
    //start new game
    newGame: function() {
      this.move = 0;
      this.board = [];
      this.left = this.boardSize * 2;
      this.hintLeft = 3;
      this.startTime = new Date().getTime();
      let arr = this.createArray();
      this.createBoard(arr);
      this.loop = setInterval(this.calculateTime, 1000);
    },
    // a function to calculate elapsed time
    calculateTime: function() {
      let currentTime = new Date().getTime();
      let elapsed = (currentTime - this.startTime) / 1000;
      this.elapsedTime = {
        day: Math.floor(elapsed / 86400),
        hour: Math.floor(elapsed / 3600 % 24),
        minute: Math.floor(elapsed / 60 % 60),
        second: Math.floor(elapsed % 60),
      };
    },
    // requiring card image from the assets directory
    getImgUrl: function(img) {
      return require('../assets/' + img + '.png');
    },
    // creates card array and suffle items
    createArray: function() {
      let arr = [...Array(this.boardSize * 4).keys()];
      for (let i = 0; i < arr.length; i += 1) {
        arr[i] += 1;
        if (arr[i] > this.boardSize * 2) arr[i] -= this.boardSize * 2;
      }
      // let's suffle the array
      arr = this.suffleArray(arr);
      return arr;
    },
    // generate board items from the array
    createBoard: function(array) {
      let row;
      for (let i = 0; i < array.length; i += 1) {
        if(i % this.boardSize === 0) row = [];
        let rowObject = { image: array[i], status: 0 };
        row.push(rowObject);
        if(i % this.boardSize === this.boardSize - 1) this.board.push(row);
      }
    },
    // a function to suffle array
    suffleArray: function(array) {
      let currentIndex = array.length;
      let temp, randomIndex;

      while (currentIndex !== 0) {
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;

        // swap elements
        temp = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temp;
      }
      return array;
    },
    // compare and decide if the picked cards are equal or not
    compareImages: function(current) {
     return current.image === this.prev.image;
    },
    // decides if the game is over
    isGameEnd: function() {
      if (this.left === 0) {
        clearInterval(this.loop);
        const text = 'Congratulations, you have completed the puzzle. Do you want to start new game?';
        const returnedValue = confirm(text);
        if(returnedValue === true) this.newGame();
      }
    },
    // show the selected card image
    showImage: function(currentRowIndex, currentColIndex) {
      this.move += 1;
      const current = this.board[currentRowIndex][currentColIndex];
      current.status = 1;
      if (this.move % 2 === 1) {
        // if there is a one selected card set it to prev
        this.prev = this.board[currentRowIndex][currentColIndex];
     } else if(this.compareImages(current)) {
       // selected two cards are equal to each other so hide them
        current.status = 2;
        this.prev.status = 2;
        this.left -= 1;
        // check if the game is over
        this.isGameEnd();
      } else {
        // selected cards are not eqaul to each other so hide them after 0.5 seconds
        setTimeout(function(current, prev) {
          current.status = 0;
          prev.status = 0;
        }, 500, current, this.prev);
      }
    },
    // showing all cards on the screen
    showCards: function() {
      let turnedCards = [];
      for (let i = 0; i < this.boardSize; i += 1) {
        for (let j = 0; j < this.boardSize; j += 1) {
          if(this.board[i][j].status !== 2) {
            turnedCards.push({ row: i, col: j });
            this.board[i][j].status = 1;
          }
        }
      }
      return turnedCards;
    },
    // hide the cards back
    hideCards: function(cards) {
      for (let i = 0; i < cards.length; i += 1) {
        this.board[cards[i].row][cards[i].col].status = 0;
      }
    },
    // a little help for the player
    showHint: function() {
      if (this.hintLeft === 0 || this.left === 0) {
        return;
      }
      this.hintLeft -= 1;
      let turnedCards = this.showCards();
      // cards are showed now. After 1 second, hide the turned cards
      setTimeout(this.hideCards, 1000, turnedCards);
    }
  },
  data() {
    return {
      move: null,
      left : null,
      boardSize: 4,
      board: null,
      prev: null,
      hintLeft: null,
      startTime: null,
      loop: null,
      elapsedTime: {},
    }
  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.header {
  margin-bottom: 50px;
}
.game-board table {
  margin: 0 auto;
  border-collapse: collapse;
}
.game-board table td {
  width: 86px;
  height: 86px;
}
.game-asset {
  width: 80px;
  height: 80px;
}
.score-board {
  border: 1px solid #ccc;
}
.score-board button {
  font-weight: bold;
  margin-right: 30px;
}
.bold {
  font-weight: bold;
}
</style>
