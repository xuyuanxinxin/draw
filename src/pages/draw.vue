<script lang="ts" setup>
import { onMounted, onUnmounted, ref, watch } from "vue"
import { ElButton } from "element-plus"

const colorInput = ref("")
const ctx = ref<CanvasRenderingContext2D>()

let x = 0
let y = 0
let isDraw = false
let shouldContinue = false
const imageSource = ref("")

const canvasContext = ref<HTMLCanvasElement>()
const draw = (e: MouseEvent) => {
  if (isDraw && canvasContext.value !== undefined) {
    const [px, py] = [
      e.clientX - canvasContext.value.offsetLeft,
      e.clientY - canvasContext.value.offsetTop,
    ]
    if ((x === 0 && y === 0) || shouldContinue) {
      ;[x, y] = [px, py]
      drawLine(x, y, x, y)
      shouldContinue = false
    } else {
      const [a, b] = [x, y]
      ;[x, y] = [px, py]
      drawLine(a, b, x, y)
    }
  }
}

function drawLine(x1: number, y1: number, x2: number, y2: number) {
  if (ctx.value !== undefined) {
    ctx.value.beginPath()
    ctx.value.moveTo(x1, y1)
    ctx.value.lineTo(x2, y2)
    ctx.value.closePath()
    ctx.value.stroke()
  }
}

const clickHandle = () => {
  isDraw = !isDraw
  if (isDraw === true) {
    shouldContinue = !shouldContinue
  }
}

// clear the canvas
const clear = () => {
  const canvasBox = canvasContext.value!
  if (ctx.value !== undefined) {
    ctx.value.clearRect(0, 0, canvasBox.width, canvasBox.height)
  }
}

watch(colorInput, () => {
  ctx.value!.strokeStyle = colorInput.value
})
const output = () => {
  const dataURL = canvasContext.value!.toDataURL("image/png", 1.0)
  imageSource.value = dataURL
  const a = document.createElement("a")
  a.href = dataURL
  a.download = "1.png"
  a.click()
  console.log(dataURL)
}

onMounted(() => {
  if (canvasContext.value !== undefined) {
    ctx.value = canvasContext.value.getContext("2d")!
    ctx.value.lineJoin = "round"
    ctx.value.lineCap = "round"
    ctx.value.lineWidth = 8
    ctx.value.fillStyle = "white"
    ctx.value.fillRect(
      0,
      0,
      canvasContext.value.width,
      canvasContext.value.height
    )
    canvasContext.value.addEventListener("mousemove", draw)
    canvasContext.value.addEventListener("click", clickHandle)
  }
})

onUnmounted(() => {
  window.removeEventListener("mousemove", draw)
  window.removeEventListener("click", clickHandle)
})
</script>

<template>
  <canvas ref="canvasContext" width="500" height="500"> </canvas>
  <div class="tool-pane">
    <el-button @click="clear">Clear</el-button>
    <el-color-picker id="colorInput" v-model="colorInput" />
    <el-button @click="output">Export</el-button>
  </div>
</template>

<style lang="scss">
canvas {
  margin-top: 150px;
  border: solid 1px black;
}
.tool-pane {
  margin: 0 auto;
  width: 500px;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}
</style>
