/* eslint-disable max-lines-per-function */
let readline = require('readline-sync');

class Square {
  static UNUSED_SQUARE = ' ';
  static HUMAN_MARKER = 'X';
  static COMPUTER_MARKER = 'O';
  static POSSIBLE_WINNING_ROWS = [
    [ "1", "2", "3" ],
    [ "4", "5", "6" ],
    [ "7", "8", "9" ],
    [ "1", "4", "7" ],
    [ "2", "5", "8" ],
    [ "3", "6", "9" ],
    [ "1", "5", "9" ],
    [ "3", "5", "7" ],
  ];

  constructor(marker = Square.UNUSED_SQUARE) {
    this.marker = marker;
  }
  toString() {
    return this.marker;
  }
  setMarker(marker) {
    this.marker = marker;
  }
  getMarker() {
    return this.marker;
  }
}

class Board {
  constructor() {
    this.clearBoard();
  }

  display() {
    console.log("");
    console.log("     |     |");
    console.log(`  ${this.squares["1"]}  |  ${this.squares["2"]}  |  ${this.squares["3"]}`);
    console.log("     |     |");
    console.log("-----+-----+-----");
    console.log("     |     |");
    console.log(`  ${this.squares["4"]}  |  ${this.squares["5"]}  |  ${this.squares["6"]}`);
    console.log("     |     |");
    console.log("-----+-----+-----");
    console.log("     |     |");
    console.log(`  ${this.squares["7"]}  |  ${this.squares["8"]}  |  ${this.squares["9"]}`);
    console.log("     |     |");
    console.log("");
  }

  markSpaceAt(choice, marker) {
    this.squares[choice].setMarker(marker);
  }

  unusedSquares(markerVal = Square.UNUSED_SQUARE) {
    let unusedSquareArray = [];
    let keys = Object.keys(this.squares);
    keys.forEach(key => {
      if (this.squares[key].toString() === markerVal) {
        unusedSquareArray.push(key);
      }
    }, this);
    return unusedSquareArray;
  }

  isFull() {
    let availableSquares = this.unusedSquares();
    return availableSquares.length === 0;
  }

  isWinner(player) {
    let arr = [];
    let rtrn = false;
    Square.POSSIBLE_WINNING_ROWS.forEach(row => {
      arr = [];
      row.forEach(val => {
        if (this.squares[val].toString() === player.getMarker()) {
          arr.push(val);
          if (arr.length === 3) {
            rtrn = true;
          }
        }
      });
    }, this);
    return rtrn;
  }

  displayWithClear() {
    console.clear();
    console.log('');
    this.display();
  }

  joinOr(arr, delimiter = ', ', word = 'or') {
    let finStr = '';
    let arr1 = [];
    arr.forEach((choice, idx) => {
      if (idx !== arr.length - 1) {
        arr1.push(choice);
      }
    });
    if (arr.length === 1) {
      finStr = String(arr[0]);
    } else if (arr.length === 2) {
      finStr = arr[0] + ` ${word} ` + arr[1];
    } else {
      finStr = arr1.join(`${delimiter}`);
      finStr += `${delimiter}` + word;
      finStr += ' ' + arr[arr.length - 1];
    }
    return finStr;
  }

  clearBoard() {
    this.squares = {};
    for (let counter = 1; counter <= 9; counter++) {
      this.squares[counter] = new Square();
    }
  }
}

class Player {
  constructor(marker) {
    this.marker = marker;
    this.score = 0;
  }

  getMarker() {
    return this.marker;
  }

  trackScore() {
    this.score += 1;
  }
}

class Human extends Player {
  constructor() {
    super(Square.HUMAN_MARKER);
  }
}

class Computer extends Player {
  constructor() {
    super(Square.COMPUTER_MARKER);
    this.moved = false;
  }
}

class TTTGame {
  constructor() {
    this.board = new Board();
    this.human = new Human();
    this.computer = new Computer();
    this.currentPlayer = this.human;
    this.humanGoesFirst = null;
  }

  play() {
    console.clear();
    this.displayWelcomeMessage();
    this.playerGoesFirst();
    this.board.display();

    while (this.human.score < 3 && this.computer.score < 3) {
      this.playOnce();
      this.board.displayWithClear();
      this.displayResults();
      if (this.human.score < 3 && this.computer.score < 3) {
        if (!this.playAgain()) break;
        this.board.clearBoard();
        this.board.displayWithClear();
      }

    }
    this.displayWinner();
    this.displayGoodbyeMessage();
  }

  playOnce() {
    while (true) {
      this.playerPlays();
      if (this.gameOver()) break;
      this.board.displayWithClear();
      this.togglePlayers();

      this.playerPlays();
      if (this.gameOver()) break;
      this.togglePlayers();
      this.board.displayWithClear();
    }
    this.swapRoundStarter();
  }

  playerPlays() {
    if (this.currentPlayer === this.human) {
      this.humanMoves();
    } else {
      this.computerMoves();
    }
  }

  swapRoundStarter() {
    if (this.humanGoesFirst) {
      this.currentPlayer = this.computer;
      this.humanGoesFirst = false;
    } else {
      this.currentPlayer = this.human;
      this.humanGoesFirst = true;
    }
  }

  playerGoesFirst() {
    this.humanGoesFirst = this.askWhoGoesFirst();
    if (!this.humanGoesFirst) {
      this.currentPlayer = this.computer;
    }
  }

  askWhoGoesFirst() {
    let answer;
    while (true) {
      console.log(`Would you like to go first? (y or n)`);
      answer = readline.question().toLowerCase();
      if (['y', 'n'].includes(answer)) break;
      console.log(`Please enter "y" or "n"`);
    }
    return answer === 'y';
  }

  togglePlayers() {
    if (this.currentPlayer === this.human) {
      this.currentPlayer = this.computer;
    } else {
      this.currentPlayer = this.human;
    }
  }

  displayWelcomeMessage() {
    console.log(`Welcome to Tic Tac Toe, score 3 points to win the match!`);
  }

  displayGoodbyeMessage() {
    console.log(`Thanks for playing Tic Tac Toe! Goodbye!`);
  }

  displayResults() {
    if (this.board.isWinner(this.human)) {
      console.log(`Human wins!`);
      this.human.trackScore();
    } else if (this.board.isWinner(this.computer)) {
      console.log(`Computer wins!`);
      this.computer.trackScore();
    } else {
      console.log(`That game was a tie.`);
    }
    console.log(`The current score is Human: ${this.human.score}. Computer: ${this.computer.score}`);
  }

  displayWinner() {
    if (this.human.score === 3) {
      console.log(`Human wins that match!`);
    } else if (this.computer.score === 3) {
      console.log(`Computer wins that match!`);
    } else {
      console.log(`There was no winner for the match.`);
    }
  }

  humanMoves() {
    let choice;
    let availableSquares = this.board.unusedSquares();
    while (true) {
      choice = readline.question(`Choose a square (${this.board.joinOr(availableSquares)}): `);
      if (availableSquares.includes(choice)) break;
      console.log('Please enter a valid choice');
    }

    this.board.markSpaceAt(choice, this.human.getMarker());
  }

  computerMoves() {
    this.computerMakesSmartMove(this.computer);
    if (!this.computer.moved) {
      this.computerMakesSmartMove(this.human);
    }
    if (!this.computer.moved) {
      this.computerPicksSquareFive();
    }
    if (!this.computer.moved) {
      this.computerMovesRandom();
    }
  }

  computerMakesSmartMove(offenseOrDefense) {
    this.computer.moved = false;
    let emptyArrSquares = this.findRiskySquares(offenseOrDefense);
    if (emptyArrSquares.length !== 0) {
      this.board.markSpaceAt(emptyArrSquares[0], this.computer.getMarker());
      this.computer.moved = true;
    }
  }

  findRiskySquares(player) {
    let arrOfTwoXs = [];
    let rowAboutToWin = [];
    let riskySquares = [];
    Square.POSSIBLE_WINNING_ROWS.forEach(row => {
      arrOfTwoXs = [];
      row.forEach(val => {
        if (this.board.squares[val].toString() === player.getMarker()) {
          arrOfTwoXs.push(val);
          if (arrOfTwoXs.length === 2) {
            rowAboutToWin.push(row);
          }
        }
      });
    });

    rowAboutToWin.forEach(row => {
      row.forEach(val => {
        if (this.board.squares[val].toString() === Square.UNUSED_SQUARE) {
          riskySquares.push(val);
        }
      });
    });
    return riskySquares;
  }

  computerPicksSquareFive() {
    this.computer.moved = false;
    let middleSquare = 5;
    if (this.board.squares[middleSquare].toString() === Square.UNUSED_SQUARE) {
      this.board.markSpaceAt(middleSquare, this.computer.getMarker());
      this.computer.moved = true;
    }
  }

  computerMovesRandom() {
    let availableSquares = this.board.unusedSquares();
    let idx = Math.floor(Math.random() * availableSquares.length);
    let choice = availableSquares[idx];
    if (!(availableSquares.length === 0)) {
      this.board.markSpaceAt(choice, this.computer.getMarker());
    }
  }

  gameOver() {
    return this.board.isFull() || this.board.isWinner(this.human) ||
    this.board.isWinner(this.computer);
  }

  playAgain() {
    return this.askToContinue();
  }

  askToContinue() {
    let choice;
    while (true) {
      choice = readline.question(`Would you like to play again (y or n)? `).toLowerCase();
      if (['y', 'n'].includes(choice)) break;
      else console.log(`Please enter either "y" or "n"`);
    }
    let continuer = false;
    if (choice === 'y') {
      continuer = true;
    }
    return continuer;
  }
}
let game = new TTTGame();
game.play();
