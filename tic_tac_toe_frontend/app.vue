<template>
  <div class="tic-tac-toe-app">
    <header class="ttt-header">
      <span class="ttt-title">Hearts & Queens Tic Tac Toe</span>
    </header>
    <main>
      <section class="ttt-board">
        <div
          v-for="(cell, i) in board"
          :key="i"
          class="ttt-cell"
          :class="{
            blink: winningCells.includes(i),
            'ttt-heart': cell === 'X',
            'ttt-queen': cell === 'O',
            disabled: isComplete || cell
          }"
          @click="makeMove(i)"
        >
          <span v-if="cell === 'X'" class="icon-heart" aria-label="Heart" />
          <span v-else-if="cell === 'O'" class="icon-queen" aria-label="Queen" />
        </div>
      </section>
      <div class="ttt-status-controls">
        <div v-if="!isComplete" class="ttt-player-turn">
          <span>Turn:</span>
          <span
            class="icon"
            :class="currentPlayer === 'X' ? 'icon-heart' : 'icon-queen'"
            :aria-label="currentPlayer === 'X' ? 'Heart' : 'Queen'"
          />
          <span class="player-label">{{ currentPlayer === 'X' ? 'Hearts' : 'Queens' }}</span>
        </div>
        <div v-if="isComplete" class="ttt-game-result">
          <span
            v-if="winner"
            class="ttt-winner"
          >
            <span v-if="winner === 'X'" class="icon-heart" aria-label="Heart" /> 
            <span v-else class="icon-queen" aria-label="Queen" />
            <span class="result-label">wins!</span>
          </span>
          <span v-else class="result-label">It's a Draw!</span>
        </div>
        <div class="ttt-controls">
          <button class="ttt-btn" @click="resetBoard" v-if="!isComplete">Reset</button>
          <button class="ttt-btn ttt-replay" v-if="isComplete" @click="resetBoard">Replay</button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
// PUBLIC_INTERFACE
/**
 * This is the main component for the Hearts & Queens Tic Tac Toe game.
 * - X is represented by a heart icon, O by a queen (crown) icon.
 * - Handles all game logic, state, animated win, and result/replay UI.
 * - Modern, responsive, light design and color palette from requirements.
 */

import { ref, computed } from 'vue'

const primary = "#d32f2f";
const secondary = "#283593";
const accent = "#ffeb3b";

const initialBoard = () => Array(9).fill("");

const board = ref<string[]>(initialBoard());
const currentPlayer = ref<"X" | "O">("X");
const isComplete = ref(false);
const winner = ref<null | "X" | "O">(null);
const winningCells = ref<number[]>([]);
const moveCount = ref(0);

// For animation
let winTimeout: ReturnType<typeof setTimeout> | null = null;

const makeMove = (idx: number) => {
  if (isComplete.value || board.value[idx]) return;
  board.value[idx] = currentPlayer.value;
  moveCount.value += 1;
  checkWinner();
  if (!isComplete.value) {
    currentPlayer.value = currentPlayer.value === "X" ? "O" : "X";
  }
};

function checkWinner() {
  const wins = [
    [0,1,2],[3,4,5],[6,7,8], // rows
    [0,3,6],[1,4,7],[2,5,8], // cols
    [0,4,8],[2,4,6]          // diags
  ];
  for (const combo of wins) {
    const [a,b,c] = combo;
    if (
      board.value[a] &&
      board.value[a] === board.value[b] &&
      board.value[a] === board.value[c]
    ) {
      isComplete.value = true;
      winner.value = board.value[a] as "X" | "O";
      winningCells.value = combo;
      // Animate winning cells (blink)
      if (winTimeout) clearTimeout(winTimeout);
      winTimeout = setTimeout(() => {
        winningCells.value = [];
      }, 1400);
      return;
    }
  }
  // Draw
  if (moveCount.value === 9) {
    isComplete.value = true;
    winner.value = null;
  }
}

function resetBoard() {
  board.value = initialBoard();
  currentPlayer.value = "X";
  isComplete.value = false;
  winner.value = null;
  winningCells.value = [];
  moveCount.value = 0;
}
</script>

<style scoped>
.tic-tac-toe-app {
  min-height: 100vh;
  background: #fafafa;
  color: #222;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: 'Montserrat', 'Segoe UI', Arial, sans-serif;
}
.ttt-header {
  width: 100%;
  min-height: 42px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 0.5px solid #e3e3e3;
  background: #fff;
  margin-bottom: 1.5rem;
  box-shadow: 0 1px 8px 0 #eee1;
}
.ttt-title {
  font-weight: bold;
  font-size: 1.5rem;
  letter-spacing: 1px;
  color: #b71c1c;
  padding: 0.6em 1.5em 0.6em 1.5em;
}
main {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.ttt-board {
  margin: 0 auto 1.5rem auto;
  display: grid;
  grid-template-columns: repeat(3, 70px);
  grid-template-rows: repeat(3, 70px);
  gap: 14px;
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 4px 36px 0 #d32f2f16, 0 2px 10px 0 #28359308;
  padding: 24px;
}
.ttt-cell {
  width: 70px;
  height: 70px;
  background: #f5f5f5;
  border-radius: 12px;
  border: 2px solid #28359333;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 2.8rem;
  transition: background 0.18s, box-shadow 0.16s;
  box-shadow: 0 1.5px 16px #d32f2f07;
  user-select: none;
  position: relative;
}
.ttt-cell:hover {
  background: #ffeb3b18;
  border-color: #d32f2f;
}
.ttt-cell.disabled {
  cursor: not-allowed;
  opacity: 0.74;
}
.icon-heart,
.ttt-heart .icon-heart {
  color: #d32f2f;
  display: inline-block;
  width: 1.65em;
  height: 1.65em;
  vertical-align: middle;
  /* Heart SVG below */
  background: none;
  mask: url('data:image/svg+xml;utf8,<svg width="48" height="48" xmlns="http://www.w3.org/2000/svg"><path fill="WHITE" d="M24 42s-12.36-7.34-17.32-12.34C2.77 26.54.17 20.64 4.05 16.1c5-5.8 13.53-5.44 17.83-.89a2.99 2.99 0 0 0 4.24 0c4.3-4.55 12.83-4.91 17.83.89 3.88 4.54 1.28 10.44-2.63 13.56C36.36 34.66 24 42 24 42z" /></svg>');
  -webkit-mask: url('data:image/svg+xml;utf8,<svg width="48" height="48" xmlns="http://www.w3.org/2000/svg"><path fill="WHITE" d="M24 42s-12.36-7.34-17.32-12.34C2.77 26.54.17 20.64 4.05 16.1c5-5.8 13.53-5.44 17.83-.89a2.99 2.99 0 0 0 4.24 0c4.3-4.55 12.83-4.91 17.83.89 3.88 4.54 1.28 10.44-2.63 13.56C36.36 34.66 24 42 24 42z" /></svg>') center/100% 100% no-repeat;
  background-color: #d32f2f;
}
.icon-queen,
.ttt-queen .icon-queen {
  color: #283593;
  display: inline-block;
  width: 1.55em;
  height: 1.55em;
  vertical-align: middle;
  background: none;
  /* Crown (queen) SVG below */
  mask: url('data:image/svg+xml;utf8,<svg width="40" height="40" xmlns="http://www.w3.org/2000/svg"><path fill="WHITE" d="M6 32l4-18 10 13 10-13 4 18H6zm30-6a3 3 0 1 1-6 0 3 3 0 0 1 6 0zm-29-3a3 3 0 1 1-6 0 3 3 0 0 1 6 0zm16-11a3 3 0 1 1-6 0 3 3 0 0 1 6 0z"/></svg>');
  -webkit-mask: url('data:image/svg+xml;utf8,<svg width="40" height="40" xmlns="http://www.w3.org/2000/svg"><path fill="WHITE" d="M6 32l4-18 10 13 10-13 4 18H6zm30-6a3 3 0 1 1-6 0 3 3 0 0 1 6 0zm-29-3a3 3 0 1 1-6 0 3 3 0 0 1 6 0zm16-11a3 3 0 1 1-6 0 3 3 0 0 1 6 0z"/></svg>') center/100% 100% no-repeat;
  background-color: #283593;
}
.blink {
  animation: ttt-blink 0.22s alternate 4;
  background: #ffeb3b4a !important;
  border-color: #ffeb3b;
  z-index: 1;
}
@keyframes ttt-blink {
  from { box-shadow: 0 0 10px 0 #ffeb3b; background: #fff;}
  to { box-shadow: 0 0 20px 8px #ffeb3b55; background: #fffde7;}
}
.ttt-status-controls {
  margin-top: 1.5rem;
  text-align: center;
  width: 100%;
  max-width: 400px;
}
.ttt-player-turn, .ttt-game-result {
  font-size: 1.21em;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 0.6em;
  margin-bottom: 1.1em;
}
.icon {
  font-size: 1.5em;
}
.player-label {
  color: #283593;
  font-weight: 600;
  margin-left: 0.2em;
  font-size: 1.1em;
}
.ttt-winner .icon-heart, .ttt-winner .icon-queen {
  font-size: 2em;
  margin-right: 0.13em;
  vertical-align: middle;
}
.result-label {
  color: #d32f2f;
  font-weight: 700;
  font-size: 1.12em;
  letter-spacing: 1px;
}
.ttt-controls {
  display: flex;
  gap: 1em;
  justify-content: center;
  margin-top: 1em;
}
.ttt-btn {
  border: none;
  outline: none;
  border-radius: 23px;
  font-size: 1.11em;
  font-family: inherit;
  color: #fff;
  background: #283593;
  padding: 0.71em 2.1em;
  letter-spacing: 0.5px;
  box-shadow: 0 1.5px 8px #28359325;
  cursor: pointer;
  transition: background 0.15s, box-shadow 0.13s;
  margin-bottom: 0.2em;
}
.ttt-btn:active, .ttt-btn:hover {
  background: #d32f2f;
}
.ttt-btn.ttt-replay {
  background: #d32f2f;
  color: #fffde7;
}
.ttt-btn.ttt-replay:active, .ttt-btn.ttt-replay:hover {
  background: #283593;
}
@media (max-width: 600px) {
  .ttt-board { grid-template-columns: repeat(3, 20vw); grid-template-rows: repeat(3, 20vw); padding: 3.5vw; gap: 3vw; }
  .ttt-cell { width: 20vw; height: 20vw; font-size: 8vw; border-radius: 6vw; }
  .ttt-header, .ttt-title { font-size: 1.1rem; padding: 0.4em 0.6em; }
}
</style>
