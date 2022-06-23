<script setup lang="ts">
const { t } = useI18n()
const state = ref()
interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}
const width = 10
const height = 10
state.value = {
  mineGenerated: false,
  status: 'ready',
  board: Array.from({ length: height }, (_, y) =>
    Array.from({ length: width },
      (_, x): BlockState => ({
        x,
        y,
        adjacentMines: 0,
        revealed: false,
        mine: false,
      }),
    ),
  ),
}
const descriptions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
function onClick(block: BlockState) {
  block.revealed = true
}
function updataNumbers() {
  state.value.board.forEach((row, y: number) => {
    row.forEach((block: BlockState, x: number) => {
      if (block.mine)
        return
      descriptions.forEach(([dx, dy]) => {
        const x2: number = x + dx
        const y2: number = y + dy
        if (x2 < 0 || x2 >= width || y2 < 0 || y2 >= height)
          return
        if (state.value.board[y2][x2].mine)
          block.adjacentMines += 1
      })
    })
  })
}
function generateMines() {
  for (const row of state.value.board) {
    for (const block of row)
      block.mine = Math.random() < 0.2
  }
}
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]
function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'
  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'

  return block.mine
    ? 'bg-red-500/50'
    : numberColors[block.adjacentMines]
}
generateMines()
updataNumbers()
</script>

<template>
  <div items-center>
    <div v-for="y, idx in state.board" :key="idx" flex justify-center>
      <template v-for="block, xidx in y" :key="xidx">
        <button border="1 gray-400/20" hover="bg-gray/20" p-2 w-10 h-10 m-1 :class="getBlockClass(block)" @click="onClick(block)">
          <div v-if="block.mine" i-mdi-mine>
            {{ 'x' }}
          </div>
          <div v-else>
            <div v-if="block.revealed">
              {{ block.adjacentMines }}
            </div>
            <div v-else>
              -
            </div>
          </div>
        </button>
      </template>
    </div>
  </div>
</template>
