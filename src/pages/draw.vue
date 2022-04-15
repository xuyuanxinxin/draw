<script lang="ts" setup>
import { onMounted, onUnmounted, ref, watch } from "vue"
import { ElButton } from "element-plus"

const colorInput = ref("")
const ctx = ref<CanvasRenderingContext2D>()

let x = 0
let y = 0
let isDraw = false
let shouldContinue = false
let isMobile = false
const imageSource = ref("")

const canvasContext = ref<HTMLCanvasElement>()
const draw = (e: MouseEvent | TouchEvent) => {
  let target: MouseEvent | Touch | null = null
  if (window.TouchEvent && e instanceof TouchEvent) {
    target = e.touches[0] as Touch
    console.log(e.touches[0])
    e.preventDefault()
    e.stopPropagation()
  } else if (e instanceof MouseEvent) {
    target = e
  }
  if (
    target !== null &&
    ((isDraw && canvasContext.value !== undefined) ||
      (isMobile && canvasContext.value !== undefined))
  ) {
    const [px, py] = [
      target.clientX - canvasContext.value.offsetLeft,
      target.clientY - canvasContext.value.offsetTop,
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
  if (isMobile) {
    shouldContinue = true
  }
}
const endHandle = () => {
  console.log("touch end")
  shouldContinue = false
}

// clear the canvas
const clear = () => {
  const canvasBox = canvasContext.value!
  if (ctx.value !== undefined) {
    ctx.value.clearRect(0, 0, canvasBox.width, canvasBox.height)
    ctx.value.fillStyle = "white"
    ctx.value.fillRect(0, 0, canvasBox.width, canvasBox.height)
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
    canvasContext.value.width = window.innerWidth
    canvasContext.value.height = window.innerHeight * 0.6
    console.log(canvasContext.value.getBoundingClientRect())

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
    canvasContext.value.addEventListener("mousedown", () => {
      isDraw = true
      shouldContinue = !shouldContinue
    })
    canvasContext.value.addEventListener("mouseup", () => {
      isDraw = false
    })

    canvasContext.value.addEventListener("mousemove", draw)
    canvasContext.value.addEventListener("touchmove", draw)
    if (
      /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
        navigator.userAgent
      )
    ) {
      isMobile = true
      canvasContext.value.addEventListener("touchstart", clickHandle)
      canvasContext.value.addEventListener("touchend", endHandle)
    } else {
      // canvasContext.value.addEventListener("click", clickHandle)
    }
  }
})

onUnmounted(() => {
  if (canvasContext.value !== undefined) {
    canvasContext.value.removeEventListener("mousemove", draw)
    canvasContext.value.removeEventListener("touchmove", draw)
    canvasContext.value.removeEventListener("click", clickHandle)
    canvasContext.value.removeEventListener("touchstart", clickHandle)
  }
})
</script>

<template>
  <canvas ref="canvasContext" />
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
  @media only screen and (min-width: 768px) {
    width: 500px;
  }
  @media only screen and (max-width: 768px) {
    width: 100%;
  }
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}
</style>
