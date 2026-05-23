<template>
  <div class="wrapper" @contextmenu.prevent>
    <div class="toolbar">
      <div class="tools">
        <button
          @click="tool = 'draw'"
          class="brushOptionButtons"
          :class="{ activeTool: tool === 'draw' }"
        >
          🖌️
        </button>

        <button
          @click="tool = 'fill'"
          class="brushOptionButtons"
          :class="{ activeTool: tool === 'fill' }"
        >
          🪣
        </button>
      </div>
      <div class="colors">
        <button
          v-for="c in colors"
          :key="c"
          class="color"
          :style="{ background: c }"
          @click="primaryColor = c"
          @mouseup.right="secondaryColor = c"
          :class="{ active: primaryColor === c }"
        />
      </div>

      <div class="secondary">
        <div
          class="color large"
          :style="{ background: primaryColor }"
        />
        <div
          class="color large"
          :style="{ background: secondaryColor }"
        />
      </div>

      <div class="slider">
        <span>Size: {{ (lineWidth < 10 ? '_'+lineWidth : lineWidth)}}</span>
        <input
          type="range"
          min="1"
          :max="MAX_LINE_WIDTH"
          v-model.number="lineWidth"
        />
      </div>

      <!-- UNDO BUTTON -->
      <button @click="undo" :disabled="history.length === 0" class="brushOptionButtons">
        🔙
      </button>
    </div>

    <canvas
      ref="canvas"
      class="board"
      @mousedown="startDraw"
      @mousemove="draw"
      @mouseup="stopDraw"
      @mouseleave="stopDraw"
      @wheel.prevent="onScroll"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const canvas = ref(null)
let ctx = null

const drawing = ref(false)
const activeButton = ref(0)
const points = ref([])

const colors = [
  '#ffffff',
  '#000000', 
  '#ff0000', 
  '#00aa00', 
  '#0000ff', 
  '#ff9900', 
  '#9900ff'
]
const primaryColor = ref('#000000')
const secondaryColor = ref('#ff0000')
const lineWidth = ref(4)
const strokeWidth = ref(4)

const tool = ref('draw') // draw, fill, erase
const MAX_LINE_WIDTH = 20

// UNDO HISTORY
const history = ref([])

onMounted(() => {
  ctx = canvas.value.getContext('2d')

  const cssWidth = 800
  const cssHeight = 600
  const dpr = window.devicePixelRatio || 1

  canvas.value.style.width = `${cssWidth}px`
  canvas.value.style.height = `${cssHeight}px`

  canvas.value.width = cssWidth * dpr
  canvas.value.height = cssHeight * dpr

  ctx.scale(dpr, dpr)
  ctx.lineCap = 'round'
  ctx.lineJoin = 'round'

  // white background
  ctx.fillStyle = 'white'
  ctx.fillRect(0, 0, cssWidth, cssHeight)

  ctx.imageSmoothingEnabled = false
})

function saveState() {
  history.value.push(
    ctx.getImageData(0, 0, canvas.value.width, canvas.value.height)
  )
}

function undo() {
  if (history.value.length === 0) return

  const previousState = history.value.pop()
  ctx.putImageData(previousState, 0, 0)
}

function onScroll(e) {
  const delta = e.deltaY > 0 ? -1 : 1
  lineWidth.value = Math.max(
    1,
    Math.min(MAX_LINE_WIDTH, lineWidth.value + delta)
  )
}


function getPos(e) {
  const rect = canvas.value.getBoundingClientRect()

  return {
    x: e.clientX - rect.left,
    y: e.clientY - rect.top
  }
}

function drawSmoothLine() {
  if (points.value.length < 2) return

  const color =
    activeButton.value === 2
      ? secondaryColor.value
      : primaryColor.value

  applyStyle(color, strokeWidth.value)

  ctx.beginPath()
  ctx.moveTo(points.value[0].x, points.value[0].y)

  for (let i = 1; i < points.value.length - 1; i++) {
    const midX = (points.value[i].x + points.value[i + 1].x) / 2
    const midY = (points.value[i].y + points.value[i + 1].y) / 2

    ctx.quadraticCurveTo(
      points.value[i].x,
      points.value[i].y,
      midX,
      midY
    )
  }

  const last = points.value[points.value.length - 1]
  ctx.lineTo(last.x, last.y)
  ctx.stroke()
}

function applyStyle(color, width = strokeWidth.value) {
  ctx.strokeStyle = color
  ctx.lineWidth = width
}
function hexToRgba(hex) {
  const bigint = parseInt(hex.slice(1), 16)

  return {
    r: (bigint >> 16) & 255,
    g: (bigint >> 8) & 255,
    b: bigint & 255,
    a: 255
  }
}

function colorsMatch(data, index, color) {
  return (
    data[index] === color.r &&
    data[index + 1] === color.g &&
    data[index + 2] === color.b &&
    data[index + 3] === color.a
  )
}

function setColor(data, index, color) {
  data[index] = color.r
  data[index + 1] = color.g
  data[index + 2] = color.b
  data[index + 3] = color.a
}

function floodFill(startX, startY, fillColorHex) {
  const dpr = window.devicePixelRatio || 1

  // convert CSS coords -> real canvas pixel coords
  startX = Math.floor(startX * dpr)
  startY = Math.floor(startY * dpr)

  const width = canvas.value.width
  const height = canvas.value.height

  const imageData = ctx.getImageData(0, 0, width, height)
  const data = imageData.data

  const fillColor = hexToRgba(fillColorHex)

  const startIndex = (startY * width + startX) * 4

  const targetColor = {
    r: data[startIndex],
    g: data[startIndex + 1],
    b: data[startIndex + 2],
    a: data[startIndex + 3]
  }

  // already filled
  if (
    targetColor.r === fillColor.r &&
    targetColor.g === fillColor.g &&
    targetColor.b === fillColor.b &&
    targetColor.a === fillColor.a
  ) {
    return
  }

  const stack = [[startX, startY]]

  while (stack.length) {
    const [x, y] = stack.pop()

    if (x < 0 || y < 0 || x >= width || y >= height) {
      continue
    }

    const index = (y * width + x) * 4

    if (!colorsMatch(data, index, targetColor)) {
      continue
    }

    setColor(data, index, fillColor)

    stack.push([x + 1, y])
    stack.push([x - 1, y])
    stack.push([x, y + 1])
    stack.push([x, y - 1])
  }

  ctx.putImageData(imageData, 0, 0)
}

function startDraw(e) {
  saveState()

  const pos = getPos(e)

  const color =
    e.button === 2
      ? secondaryColor.value
      : primaryColor.value

  // FILL TOOL
  if (tool.value === 'fill') {
    floodFill(pos.x, pos.y, color
    )
    return
  }

  // DRAW TOOL
  drawing.value = true
  activeButton.value = e.button
  strokeWidth.value = lineWidth.value

  points.value = [pos]

  applyStyle(color, strokeWidth.value)
}

function draw(e) {
  if (!drawing.value) return

  points.value.push(getPos(e))
  drawSmoothLine()
}

function stopDraw() {
  drawing.value = false
  points.value = []
}
</script>



<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  gap: 10px;
  height: 700px;
  width: 900px;
  overflow-y: auto;
  background-color: #3d3d36;

  justify-content: center; /* vertical axis (main axis) */
  align-items: center;     /* horizontal axis (cross axis) */
}


.board {
  border: 1px solid #ccc;
  cursor: crosshair;
  background: white;
}

.brushOptionButtons {
  display: flex;
  gap: 20px;
  height: 50px;
  width: 50px;
  position: relative;
  font-size: 30px;
  background-color: rgb(211, 211, 211);
  justify-content: center; /* vertical axis (main axis) */
  align-items: center;     /* horizontal axis (cross axis) */
  color: white;
}

.toolbar {
  display: flex;
  gap: 20px;
  height: 70px;
  align-items: center;
  flex-wrap: wrap;
}


.colors {
  display: flex;
  gap: 6px;
  width: 200px;
  background-color: rgb(83, 215, 255);
  height: 30px;

  justify-content: center; /* vertical axis (main axis) */
  align-items: center;     /* horizontal axis (cross axis) */
}


.color {
  width: 22px;
  height: 22px;
  border: 2px solid transparent;
  cursor: pointer;
}


.color.active {
  border-color: #333;
}


.color.large {
  width: 30px;
  height: 30px;
}


.slider {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 300px;
  color:white;
}

.tools {
  display: flex;
  gap: 10px;
}

.activeTool {
  border: 2px solid #333;
  font-weight: bold;
}
</style>