<template>
  <div>
    <button @click="startGame()">Start Game</button>
    <p class="message">{{message}}</p>

    <div :class="'board ' + (isGameActive ? 'active' : '')">
      <div
        v-for="(column, index) in board.columns"
        :key="index"
        @click="placePiece(index)"
        class="board__column"
      >
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[5]"></div>
        </div>
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[4]"></div>
        </div>
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[3]"></div>
        </div>
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[2]"></div>
        </div>
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[1]"></div>
        </div>
        <div class="board__column__cell">
          <div :class="'board__column__cell--inner '+column[0]"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";

export default {
  name: "Board",
  data() {
    return {
      isGameActive: false,
      message: "Waiting for game to start...",
      board: { columns: [] },
      // Red vs. Blue; Flag for next player
      isRedNext: true,
    }
  },
  methods: {
    startGame() {
      this.board = { columns: [[], [], [], [], [], [], []] };
      this.isGameActive = true;
      this.message = "Game started! Red to move...";
    },
    placePiece(index) {
      if (this.isGameActive) {
        const col = this.board.columns[index];
        if (col.length < 6) {
          const color = this.isRedNext ? 'Red' : 'Blue';
          col.push(color);
          if (this.checkWinCondition(index, color)) {
            this.message = `${color} wins!`;
            this.isGameActive = false;
          } else {
            this.isRedNext = !this.isRedNext;
            this.message = `${this.isRedNext ? 'Red' : 'Blue'} to move...`;
          }
        }
      }
    },
    checkWinCondition(colIndex, color) {
      if (this.isGameActive) {
        // colIndex gives the column of last move, check starting from there
        const winCondition = [color, color, color, color];

        // 4 in a column
        const col = this.board.columns[colIndex];
        if (_.isEqual(col.slice(-4), winCondition)) {
          return true;
        }

        // Get min and max indexes to look for 4 in a row
        const minColIndex = Math.max(colIndex-3, 0);
        const maxColIndex = Math.min(colIndex+3, 6);
        // Get index of the row where the piece was placed
        const rowIndex = this.board.columns[colIndex].length - 1;

        // 4 in a row
        const row = [];
        this.board.columns.map(column => row.push(column[rowIndex]));
        for (let i = minColIndex; i <= minColIndex+3 && i <= maxColIndex-3; i++) {
          const fourToCheck = row.slice(i, i+4);
          if (_.isEqual(fourToCheck, winCondition)) {
            return true;
          }
        }

        // 4 in a diagonal
        // TODO: Ever heard of elegance?
        for (let i = minColIndex; i <= minColIndex+3 && i <= maxColIndex-3; i++) {
          const colOffset = Math.abs(colIndex-i);
          // TODO: Probably need to add an inner loop to construct the arrays correctly
          const diagonalUpLeft = [
            this.board.columns[i] ? this.board.columns[i][rowIndex+colOffset] : null,
            this.board.columns[i+1] ? this.board.columns[i+1][rowIndex+colOffset-1] : null,
            this.board.columns[i+2] ? this.board.columns[i+2][rowIndex+colOffset-2] : null,
            this.board.columns[i+3] ? this.board.columns[i+3][rowIndex+colOffset-3] : null
          ];
          if (_.isEqual(diagonalUpLeft, winCondition)) {
            return true;
          }
          const diagonalDownLeft = [
            this.board.columns[i] ? this.board.columns[i][rowIndex-(colOffset)] : null,
            this.board.columns[i+1] ? this.board.columns[i+1][rowIndex-(colOffset-1)] : null,
            this.board.columns[i+2] ? this.board.columns[i+2][rowIndex-(colOffset-2)] : null,
            this.board.columns[i+3] ? this.board.columns[i+3][rowIndex-(colOffset-3)] : null
          ];
          if (_.isEqual(diagonalDownLeft, winCondition)) {
            return true;
          }
        }

        // Tie if board is filled
        const tie = this.board.columns.reduce((acc, column) => acc && column.length === 6, true);
        if (tie) {
          this.message = "Tie game!";
          this.isGameActive = false;
        }
      }

      return false;
    }
  }
}
</script>

<style lang="scss" scoped>
  $border: 1px solid #ccc;

  /* Hold space if message is empty */
  /* TODO: This should really be handled by the parent container */
  p.message {
    min-height: 1em;
    line-height: 1em;
  }

  .board {
    display: flex;
    justify-content: center;

    &__column {
      cursor: pointer;
      border-bottom: $border;

      &:last-child {
        border-right: $border;
      }

      &__cell {
        height: 60px;
        width: 60px;
        border-top: $border;
        border-left: $border;

        &--inner {
          height: 80%;
          width: 80%;
          margin-top: 10%;
          margin-left: 10%;
          border-radius: 30px;
          border: $border;

          &.Red {
            background-color: #f00;
          }
          &.Blue {
            background-color: #00f;
          }
        }
      }
    }

    &.active .board__column:hover {
      background-color: #bbb;
    }
  }
</style>