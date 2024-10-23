<script lang="ts" setup>
import html2canvas from 'html2canvas'
import { computed, ref, watch } from 'vue'

const grid = ref<number>(8)
const selectedColor = ref<string>('#000000')
const gridElement = ref<HTMLElement | null>(null)
const tipsVisible = ref(true)
const colorPalette = ref<string[]>([
  '#000000',
  '#4b4b4b',
  '#ff0000',
  '#00ff00',
  '#0000ff',
  '#ffff00',
  '#ffd700',
  '#ffa500',
  '#ff4500',
  '#9932cc',
])

const gridData = ref<string[][]>(
  Array.from({ length: grid.value }, () => Array(grid.value).fill('#ffffff')),
)

const handleClick = (row: number, col: number) => {
  handleColorChange(selectedColor.value, row, col)
}

const handleColorChange = (color: string, row: number, col: number) => {
  selectedColor.value = color
  gridData.value[row][col] = color
}

const itemColor = computed(() => (row: number, col: number) => {
  if (gridData.value && gridData.value[row] && gridData.value[row][col]) {
    return gridData.value[row][col]
  }
  return '#ffffff'
})

const selectColor = (color: string) => {
  selectedColor.value = color
}

const downloadImage = async () => {
  try {
    if (!gridElement.value) {
      throw new Error('Grid element not found')
    }

    const canvas = await html2canvas(gridElement.value, {
      backgroundColor: null,
      scale: 1,
    })

    const dataUrl = canvas.toDataURL('image/png')
    const link = document.createElement('a')
    link.href = dataUrl
    link.download = 'pixel-art.png'
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
  } catch (error) {
    console.error('Error generating image:', error)
    alert('Error generating image')
  }
}

watch(grid, () => {
  gridData.value = Array.from({ length: grid.value }, () =>
    Array(grid.value).fill('#ffffff'),
  )
})

const isPainting = ref(false)

const startPainting = () => {
  isPainting.value = true
}

const stopPainting = () => {
  isPainting.value = false
}

const paint = (row: number, col: number) => {
  if (isPainting.value) {
    handleColorChange(selectedColor.value, row, col)
  }
}

const closeTips = () => {
  tipsVisible.value = false
}
</script>

<template>
  <div class="tips" v-if="tipsVisible">
    <div class="tips-content">
      <span class="tips-close" @click="closeTips">Close (x)</span>
      <span class="tips-title">Tips</span>
      <ul>
        <li>Select the grid size</li>
        <li>
          Click on the grid to paint with the selected color. Hold the mouse
          button down to paint while moving the mouse.
        </li>
        <li>
          Use the color picker to select a custom color. Click on the grid to
          paint with the custom color.
        </li>
        <li>Download the image by clicking the "Download" button.</li>
        <li>Try it on mobile!</li>
        <li>Try to make a pixel art!</li>
      </ul>
    </div>
  </div>
  <div class="container">
    <h1>Pixel Art</h1>
    <div class="grid-size-selector">
      <label for="grid">Select the grid size</label>
      <select id="grid" v-model="grid">
        <option :value="8">8</option>
        <option :value="12">12</option>
        <option :value="16">16</option>
        <option :value="32">32</option>
      </select>
    </div>
    <div class="color-palette">
      <div
        v-for="(color, index) in colorPalette"
        @click="selectColor(color)"
        :key="index"
        class="color-item"
        :class="{ active: selectedColor === color }"
        :style="{ backgroundColor: color }"
      ></div>
    </div>
    <div class="color-picker-container">
      <label for="color-picker">Select custom color</label>
      <input class="color-picker" type="color" v-model="selectedColor" />
    </div>
    <div
      class="grid"
      ref="gridElement"
      @mousedown="startPainting"
      @mouseup="stopPainting"
      @mouseleave="stopPainting"
    >
      <div v-for="(_, rowIndex) in grid" :key="rowIndex" class="grid-row">
        <div
          v-for="(_, colIndex) in grid"
          :key="colIndex"
          class="grid-item"
          @mousedown="handleClick(rowIndex, colIndex)"
          @mousemove="paint(rowIndex, colIndex)"
          :style="{ backgroundColor: itemColor(rowIndex, colIndex) }"
        ></div>
      </div>
    </div>
    <button class="download-button" @click="downloadImage">
      Download your pixel art!
    </button>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.grid {
  display: flex;
  flex-direction: column;
  max-width: 600px;
  width: 100%;
  border: 1px solid black;
}

.grid-row {
  display: flex;
  flex: 1;
}

.grid-item {
  flex: 1;
  border: 1px solid black;
  box-sizing: border-box;
  cursor: pointer;
  aspect-ratio: 1/1;
}

.grid-size-selector {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 10px;
  margin-bottom: 16px;
}

.grid-size-selector select {
  padding: 5px 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.color-palette {
  display: flex;
  flex-direction: row;
  gap: 10px;
  margin-bottom: 16px;
  max-width: 600px;
  width: 100%;
  justify-content: center;
}

.color-item {
  width: 50px;
  cursor: pointer;
  border: 1px solid black;
  aspect-ratio: 1/1;
}

.color-item.active {
  border: 2px solid black;
}

.color-picker-container {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-bottom: 16px;
  gap: 10px;
}

input[type='color'] {
  -webkit-appearance: none;
  border: none;
}
input[type='color']::-webkit-color-swatch-wrapper {
  padding: 0;
}
input[type='color']::-webkit-color-swatch {
  border: none;
}

.tips {
  background-color: #fff;
  padding: 10px;
  background-color: #f8c408;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.tips-content {
  position: relative;
}

.tips-title {
  font-size: 1.2rem;
  font-weight: bold;
}

.tips-content ul {
  padding-left: 20px;
  margin: 0;
  margin-top: 10px;
}

.tips-close {
  cursor: pointer;
  position: absolute;
  top: 0;
  right: 0;
}

.download-button {
  margin-top: 16px;
  margin-bottom: 50px;
  border: none;
  background-color: #1dc45f;
  color: #fff;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
}
</style>
