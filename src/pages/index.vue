<script setup lang="ts">
import type { BlockState } from '~/types'
const { t } = useI18n()
const state = ref()
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
let mineGenerated: Boolean = false
const dev: Boolean = true
function onReightClick(block: BlockState): void {
  if (!block.revealed)
    block.flagged = !block.flagged
}
watch(state, checkGameState, { deep: true })
function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  if (block.mine) {
    alert('over!')
    return
  }
  block.revealed = true

  expendZero(block)
}
function updataNumbers() {
  state.value.board.forEach((row, y: number) => {
    row.forEach((block: BlockState, x: number) => {
      if (block.mine)
        return
      getSiblings(block).forEach((el) => {
        if (el.mine)
          block.adjacentMines += 1
      })
    })
  })
}
function getSiblings(block: BlockState) {
  return descriptions.map(([dx, dy]) => {
    const x2: number = block.x + dx
    const y2: number = block.y + dy
    if (x2 < 0 || x2 >= width || y2 < 0 || y2 >= height)
      return undefined
    return state.value.board[y2][x2]
  }).filter(Boolean) as BlockState[]
}
function generateMines(row: BlockState) {
  for (const row of state.value.board) {
    for (const block of row) {
      if (Math.abs(row.x - block.x) < 1 || Math.abs(row.y - block.y) < 1)
        continue
      block.mine = Math.random() < 0.2
    }
  }
  updataNumbers()
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

function expendZero(block: BlockState) {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((el) => {
    if (!el.revealed) {
      el.revealed = true
      expendZero(el)
    }
  })
}
function checkGameState() {
  if (!mineGenerated)
    return
  console.log(123)

  const blocks = state.value.board.flat()
  if (blocks.every(el => el.revealed || el.flagged)) {
    if (blocks.some(el => el.flagged && !el.mine))
      alert('You cheat')
    else
      alert('You win!')
  }
}
</script>

<template>
  <h2>Mine-clearance</h2>
  <div items-center>
    <div v-for="y, idx in state.board" :key="idx" flex justify-center>
      <template v-for="block, xidx in y" :key="xidx">
        <button border="1 gray-400/20" hover="bg-gray/20" p-2 w-10 h-10 m-1 :class="getBlockClass(block)" @click="onClick(block)" @contextmenu.prevent="onReightClick(block)">
          <template v-if="block.flagged">
            <div i-mdi-flag text-red />
          </template>
          <template v-else>
            <div v-if="block.mine && block.revealed" i-mdi-mine>
              {{ 'x' }}
            </div>
            <div v-else>
              <div v-if="block.revealed">
                {{ block.adjacentMines }}
              </div>
              <div v-else />
            </div>
          </template>
        </button>
      </template>
    </div>
  </div>
</template>
