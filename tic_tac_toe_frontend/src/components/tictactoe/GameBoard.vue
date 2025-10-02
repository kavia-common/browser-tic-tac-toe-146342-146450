<script setup lang="ts">
import { computed, reactive, ref } from 'vue'

type Player = 'X' | 'O'
type Cell = Player | ''

// PUBLIC_INTERFACE
function calculateWinner(squares: Cell[]): { winner: Player | null; line: number[] } {
  /** Determine the winner and the winning line for a 3x3 Tic Tac Toe board. */
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8], // rows
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8], // cols
    [0, 4, 8],
    [2, 4, 6], // diags
  ]
  for (const [a, b, c] of lines) {
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return { winner: squares[a] as Player, line: [a, b, c] }
    }
  }
  return { winner: null, line: [] }
}

// PUBLIC_INTERFACE
function isBoardFull(squares: Cell[]): boolean {
  /** Check if all cells are filled. */
  return squares.every((c) => c !== '')
}

const state = reactive({
  board: Array<Cell>(9).fill(''),
  xIsNext: true,
  scores: {
    X: 0,
    O: 0,
    draws: 0,
  },
  gameOver: false,
  winningLine: [] as number[],
})

const currentPlayer = computed<Player>(() => (state.xIsNext ? 'X' : 'O'))

const result = computed(() => {
  const res = calculateWinner(state.board)
  const winner = res.winner
  if (winner) {
    return { status: 'win', winner, line: res.line }
  }
  if (isBoardFull(state.board)) {
    return { status: 'draw' as const }
  }
  return { status: 'ongoing' as const }
})

const statusText = computed(() => {
  if (result.value.status === 'win') {
    return `Player ${result.value.winner} wins!`
  }
  if (result.value.status === 'draw') {
    return `It's a draw.`
  }
  return `Player ${currentPlayer.value}'s turn`
})

const statusAccent = computed(() => {
  // color accents for status chip
  if (result.value.status === 'win') return 'win'
  if (result.value.status === 'draw') return 'draw'
  return currentPlayer.value === 'X' ? 'x' : 'o'
})

function handleCellClick(index: number) {
  if (state.board[index] !== '' || state.gameOver) return

  state.board[index] = currentPlayer.value
  const evalResult = calculateWinner(state.board)

  if (evalResult.winner) {
    state.gameOver = true
    state.winningLine = evalResult.line
    state.scores[evalResult.winner] += 1
    return
  }

  if (isBoardFull(state.board)) {
    state.gameOver = true
    state.scores.draws += 1
    return
  }

  state.xIsNext = !state.xIsNext
}

function newRound() {
  state.board = Array<Cell>(9).fill('')
  state.xIsNext = !state.gameOver ? state.xIsNext : true // default back to X if game ended
  state.gameOver = false
  state.winningLine = []
}

function resetAll() {
  state.board = Array<Cell>(9).fill('')
  state.xIsNext = true
  state.gameOver = false
  state.winningLine = []
  state.scores.X = 0
  state.scores.O = 0
  state.scores.draws = 0
}

function isWinningCell(idx: number) {
  return state.winningLine.includes(idx)
}

const hoverIndex = ref<number | null>(null)
</script>

<template>
  <section class="card">
    <div class="scoreboard">
      <div class="score score-x" :class="{ active: !state.gameOver && currentPlayer === 'X' }">
        <div class="label">Player X</div>
        <div class="value">{{ state.scores.X }}</div>
      </div>
      <div class="score score-draw">
        <div class="label">Draws</div>
        <div class="value">{{ state.scores.draws }}</div>
      </div>
      <div class="score score-o" :class="{ active: !state.gameOver && currentPlayer === 'O' }">
        <div class="label">Player O</div>
        <div class="value">{{ state.scores.O }}</div>
      </div>
    </div>

    <div class="status-row">
      <div class="chip" :data-accent="statusAccent">
        <span class="dot"></span>
        {{ statusText }}
      </div>

      <div class="actions">
        <button class="btn secondary" @click="newRound" :disabled="state.board.every(c => c === '')">
          New Round
        </button>
        <button class="btn primary" @click="resetAll">
          Reset All
        </button>
      </div>
    </div>

    <div
      class="board"
      role="grid"
      aria-label="Tic Tac Toe Board"
    >
      <button
        v-for="(cell, idx) in state.board"
        :key="idx"
        class="cell"
        role="gridcell"
        :aria-label="cell ? ('Cell ' + (idx + 1) + ' ' + cell) : ('Cell ' + (idx + 1) + ' empty')"
        :data-player="cell || undefined"
        :data-hover="(!cell && !state.gameOver) ? currentPlayer : undefined"
        :class="{ win: isWinningCell(idx), disabled: !!cell || state.gameOver }"
        @click="handleCellClick(idx)"
        @mouseenter="hoverIndex = idx"
        @mouseleave="hoverIndex = null"
      >
        <span class="mark" :class="{ show: !!cell }">
          {{ cell }}
        </span>
      </button>
    </div>
  </section>
</template>

<style scoped>
/* Theme tokens (Ocean Professional) */
:root {
  --ocean-primary: #2563EB; /* blue */
  --ocean-primary-600: #1d4ed8;
  --ocean-secondary: #F59E0B; /* amber */
  --ocean-error: #EF4444;
  --ocean-bg: #f9fafb; /* gray-50 */
  --ocean-surface: #ffffff;
  --ocean-text: #111827; /* gray-900 */
  --ocean-muted: #6b7280; /* gray-500 */
  --ocean-border: #e5e7eb; /* gray-200 */
  --ocean-shadow: 0 10px 25px rgba(17, 24, 39, 0.05), 0 2px 10px rgba(17, 24, 39, 0.05);
  --ocean-radius: 16px;
}

/* Card container */
.card {
  width: 100%;
  max-width: 720px;
  background: var(--ocean-surface);
  border: 1px solid var(--ocean-border);
  border-radius: var(--ocean-radius);
  box-shadow: var(--ocean-shadow);
  padding: 1.25rem;
  backdrop-filter: saturate(120%) blur(2px);
}

/* Scoreboard */
.scoreboard {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 0.75rem;
  align-items: stretch;
}

.score {
  background: linear-gradient(180deg, rgba(37,99,235,0.06), rgba(255,255,255,1));
  border: 1px solid var(--ocean-border);
  border-radius: 12px;
  padding: 0.85rem 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  transition: transform .2s ease, box-shadow .2s ease, border-color .2s ease;
}

.score .label {
  font-size: 0.9rem;
  color: var(--ocean-muted);
}

.score .value {
  font-size: 1.4rem;
  font-weight: 700;
  color: var(--ocean-text);
}

.score.active {
  border-color: rgba(37,99,235,0.35);
  box-shadow: 0 6px 18px rgba(37,99,235,0.12);
  transform: translateY(-2px);
}

.score-draw {
  min-width: 120px;
  text-align: center;
}

.score-x .label { color: var(--ocean-primary-600); }
.score-o .label { color: var(--ocean-secondary); }

/* Status and Actions */
.status-row {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  justify-content: space-between;
  margin: 1rem 0 1.25rem;
  flex-wrap: wrap;
}

.chip {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem .8rem;
  border-radius: 999px;
  font-weight: 600;
  font-size: 0.95rem;
  color: var(--ocean-text);
  background: #f3f4f6;
  border: 1px solid var(--ocean-border);
  box-shadow: 0 2px 8px rgba(17,24,39,0.06) inset;
}

.chip .dot {
  width: 8px;
  height: 8px;
  border-radius: 999px;
  background: var(--ocean-primary);
  box-shadow: 0 0 0 4px rgba(37,99,235,0.15);
  transition: background-color .3s ease, box-shadow .3s ease;
}

.chip[data-accent="x"] .dot {
  background: var(--ocean-primary);
  box-shadow: 0 0 0 4px rgba(37,99,235,0.15);
}
.chip[data-accent="o"] .dot {
  background: var(--ocean-secondary);
  box-shadow: 0 0 0 4px rgba(245,158,11,0.18);
}
.chip[data-accent="win"] .dot {
  background: #10b981;
  box-shadow: 0 0 0 4px rgba(16,185,129,0.18);
}
.chip[data-accent="draw"] .dot {
  background: #6b7280;
  box-shadow: 0 0 0 4px rgba(107,114,128,0.18);
}

.actions {
  display: flex;
  gap: 0.5rem;
}

.btn {
  appearance: none;
  border: 1px solid var(--ocean-border);
  border-radius: 10px;
  padding: 0.55rem 0.9rem;
  font-weight: 600;
  font-size: 0.95rem;
  color: var(--ocean-text);
  background: white;
  box-shadow: var(--ocean-shadow);
  cursor: pointer;
  transition: transform .15s ease, box-shadow .2s ease, background-color .2s ease, border-color .2s ease;
}
.btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 12px 22px rgba(17,24,39,0.08);
}
.btn:active {
  transform: translateY(0);
}
.btn:disabled {
  opacity: .6;
  cursor: not-allowed;
}

.btn.primary {
  background: linear-gradient(180deg, rgba(37,99,235,0.1), rgba(37,99,235,0.08));
  border-color: rgba(37,99,235,0.35);
  color: #1f2937;
}
.btn.primary:hover {
  border-color: rgba(37,99,235,0.6);
}

.btn.secondary {
  background: linear-gradient(180deg, rgba(245,158,11,0.1), rgba(245,158,11,0.08));
  border-color: rgba(245,158,11,0.35);
  color: #1f2937;
}
.btn.secondary:hover {
  border-color: rgba(245,158,11,0.6);
}

/* Board */
.board {
  display: grid;
  gap: 10px;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  width: 100%;
  max-width: 420px;
  margin: 0 auto;
  padding: 10px;
  border-radius: 16px;
  background: linear-gradient(135deg, rgba(37,99,235,0.06), rgba(249,250,251,1));
  border: 1px solid var(--ocean-border);
  box-shadow: var(--ocean-shadow);
}

.cell {
  aspect-ratio: 1 / 1;
  border-radius: 14px;
  background: white;
  border: 1px solid var(--ocean-border);
  display: grid;
  place-items: center;
  font-size: 2.4rem;
  font-weight: 800;
  color: var(--ocean-text);
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: transform .1s ease, box-shadow .2s ease, border-color .2s ease, background-color .2s ease;
  box-shadow: 0 2px 10px rgba(17,24,39,0.05);
  user-select: none;
}
.cell:hover:not(.disabled) {
  transform: translateY(-2px);
  box-shadow: 0 12px 22px rgba(17,24,39,0.08);
  border-color: rgba(37,99,235,0.35);
  background: linear-gradient(0deg, rgba(37,99,235,0.04), rgba(255,255,255,1));
}
.cell:active:not(.disabled) {
  transform: translateY(0);
}

.cell[data-hover="X"]::after,
.cell[data-hover="O"]::after {
  content: attr(data-hover);
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  color: rgba(31,41,55,0.12);
  font-size: 2rem;
  font-weight: 900;
  pointer-events: none;
}

.cell[data-player="X"] {
  border-color: rgba(37,99,235,0.55);
}
.cell[data-player="O"] {
  border-color: rgba(245,158,11,0.6);
}

.cell.win {
  background: linear-gradient(180deg, rgba(16,185,129,0.14), rgba(255,255,255,1));
  border-color: rgba(16,185,129,0.6);
  box-shadow: 0 10px 26px rgba(16,185,129,0.22);
}

.cell.disabled {
  cursor: default;
}

.mark {
  opacity: 0;
  transform: scale(0.9);
  transition: opacity .18s ease, transform .18s ease;
}
.mark.show {
  opacity: 1;
  transform: scale(1);
}

@media (max-width: 520px) {
  .scoreboard {
    grid-template-columns: 1fr 1fr;
  }
  .score-draw {
    grid-column: span 2;
  }
  .actions {
    width: 100%;
    justify-content: stretch;
  }
  .actions .btn {
    flex: 1;
  }
}
</style>
