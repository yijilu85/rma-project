<script setup>
import { onMounted, ref, toRefs } from 'vue'

const props = defineProps({ sourceImage: Object })

const { sourceImage } = toRefs(props)

const closeModal = function () {
  const files = fileInputRef.value.files
  if (files.length <= 0) {
    return false
  }
}

var canvas
var context
var rasterSlider
var greyscaleSlider
var flippedHorizonally = false
var flippedVertically = false
var img = new Image()
img.crossOrigin = `Anonymous`

const raster = function () {
  context.clearRect(0, 0, canvas.width, canvas.height)
  context.drawImage(img, 0, 0)
  var imgData = context.getImageData(0, 0, canvas.width, canvas.height)

  var rasterSize = rasterSlider.value
  var greyscaleFactor = (greyscaleSlider.value / 100) * -1

  for (var x = 0; x < imgData.width; x++) {
    for (var y = 0; y < imgData.height; y++) {
      var rasterX = ((x / rasterSize) | 0) * rasterSize
      var rasterY = ((y / rasterSize) | 0) * rasterSize

      var rasterValIndex = (rasterX + rasterY * imgData.width) * 4

      let r = imgData.data[rasterValIndex]
      let g = imgData.data[rasterValIndex + 1]
      let b = imgData.data[rasterValIndex + 2]
      let a = imgData.data[rasterValIndex + 3]

      let CurrentPixelIndex = (x + y * imgData.width) * 4
      imgData.data[CurrentPixelIndex] = r
      imgData.data[CurrentPixelIndex + 1] = g
      imgData.data[CurrentPixelIndex + 2] = b
      imgData.data[CurrentPixelIndex + 3] = a
    }
  }

  for (var x = 0; x < imgData.width; x++) {
    for (var y = 0; y < imgData.height; y++) {
      var index = (x + y * imgData.width) * 4

      let r = imgData.data[index]
      let g = imgData.data[index + 1]
      let b = imgData.data[index + 2]

      let hsl = rgbToHsl(r, g, b)

      hsl[1] *= greyscaleFactor

      let rgb = hslToRgb(hsl[0], hsl[1], hsl[2])

      imgData.data[index] = rgb[0]
      imgData.data[index + 1] = rgb[1]
      imgData.data[index + 2] = rgb[2]
    }
  }
  if (flippedHorizonally) {
    imgData = flipHorinzotally(imgData)
  }
  if (flippedVertically) {
    imgData = flipVertically(imgData)
  }
  context.putImageData(imgData, 0, 0)
}

const flipHorinzotally = function (imgData) {
  var flippedData = imgData

  for (let x = 0; x < flippedData.width / 2; x++) {
    for (let y = 0; y < flippedData.height; y++) {
      const currentPixel = (x + y * flippedData.width) * 4
      const mirrorPixel = (flippedData.width - x - 1 + y * flippedData.width) * 4
      const tempStore = []

      for (let i = 0; i < 4; i++) {
        tempStore[i] = flippedData.data[mirrorPixel + i]
      }

      for (let i = 0; i < 4; i++) {
        flippedData.data[mirrorPixel + i] = flippedData.data[currentPixel + i]
      }

      for (let i = 0; i < 4; i++) {
        flippedData.data[currentPixel + i] = tempStore[i]
      }
    }
  }
  return flippedData
}

const setFLipHorizontal = function () {
  flippedHorizonally = !flippedHorizonally
  raster()
}
const setFLipVertical = function () {
  flippedVertically = !flippedVertically
  raster()
}
const flipVertically = function (imgData) {
  var flippedData = imgData

  for (let y = 0; y < flippedData.height / 2; y++) {
    for (let x = 0; x < flippedData.width; x++) {
      const currentPixel = (x + y * flippedData.width) * 4
      const mirrorPixel = (flippedData.width * (flippedData.height - 1 - y) + x) * 4
      const tempStore = []

      for (let i = 0; i < 4; i++) {
        tempStore[i] = flippedData.data[mirrorPixel + i]
      }

      for (let i = 0; i < 4; i++) {
        flippedData.data[mirrorPixel + i] = flippedData.data[currentPixel + i]
      }

      for (let i = 0; i < 4; i++) {
        flippedData.data[currentPixel + i] = tempStore[i]
      }
    }
  }
  return flippedData
}

const downloadImage = function () {
  var dataURL = canvas.toDataURL('image/png')
  var a = document.createElement('a')
  a.href = dataURL
  a.download = 'canvas-download.jpeg'
  a.click()
}

const loadCanvas = function () {
  canvas = document.getElementById('canvas')
  context = canvas.getContext('2d')
  canvas.width = img.width
  canvas.height = img.height
  rasterSlider = document.getElementById('rasterSlider')
  greyscaleSlider = document.getElementById('greyscaleSlider')
  raster()
}

const loadImg = function () {
  img.src = sourceImage.value.src
  img.addEventListener('load', function () {
    loadCanvas()
  })
}

const hslToRgb = function (h, s, l) {
  let r, g, b

  if (s === 0) {
    r = g = b = l
  } else {
    let hue2rgb = function (p, q, t) {
      if (t < 0) t += 1
      if (t > 1) t -= 1
      if (t < 1 / 6) return p + (q - p) * 6 * t
      if (t < 1 / 2) return q
      if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6
      return p
    }

    let q = l < 0.5 ? l * (1 + s) : l + s - l * s
    let p = 2 * l - q

    r = hue2rgb(p, q, h + 1 / 3)
    g = hue2rgb(p, q, h)
    b = hue2rgb(p, q, h - 1 / 3)
  }

  return [r * 255, g * 255, b * 255]
}
const rgbToHsl = function (r, g, b) {
  r /= 255
  g /= 255
  b /= 255

  let max = Math.max(r, g, b),
    min = Math.min(r, g, b)
  let h,
    s,
    l = (max + min) / 2

  if (max === min) {
    h = s = 0
  } else {
    let d = max - min
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
    switch (max) {
      case r:
        h = (g - b) / d + (g < b ? 6 : 0)
        break
      case g:
        h = (b - r) / d + 2
        break
      case b:
        h = (r - g) / d + 4
        break
    }
    h /= 6
  }
  return [h, s, l]
}

const emit = defineEmits(['singleImageLoaded'])
</script>

<template>
  <div class="lightbox">
    <div class="single-container">
      <div class="header">
        <h1 v-if="sourceImage">{{ sourceImage.name }}</h1>
        <button @click="$emit('closeModal')" class="closeButton button-flex">
          Close <img class="feather-white" src="../assets/x-circle.svg" />
        </button>
      </div>
      <div class="img-controls">
        <div class="sliders">
          <label name="">Verpixelung:</label>
          <input @change="raster" type="range" id="rasterSlider" min="1" max="40" value="0" />
        </div>
        <div class="sliders">
          <label name="">Graustufen:</label>
          <input
            @change="raster"
            type="range"
            id="greyscaleSlider"
            min="-100"
            max="0"
            value="-100"
          />
        </div>
        <button @click="setFLipHorizontal" class="img-control-button button-flex button-primary">
          Flip <img class="feather-white small" src="../assets/repeat.svg" />
        </button>
        <button @click="setFLipVertical" class="img-control-button button-flex button-primary">
          Flip <img class="feather-white rotate small" src="../assets/repeat.svg" />
        </button>
        <button @click="downloadImage" class="img-control-button button-flex button-primary">
          Download <img class="feather-white small" src="../assets/download.svg" />
        </button>
      </div>
      <div class="img-container">
        <canvas id="canvas" width="500" height="500">
          Sorry, your browser does not support the HTML 5 Canvas.
        </canvas>
      </div>
      <img :src="sourceImage.src" class="hidden" alt="" @load="loadImg" />
    </div>
  </div>
</template>

<style scoped>
.single-container {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  gap: 15px;
  align-items: space-between;
}

.small {
  width: 20px;
  height: 20px;
}
.img-container {
  display: flex;
  flex-flow: column nowrap;
  align-items: center;
}
div.img-controls {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  gap: 15px;
  align-items: center;
}

div.img-controls label {
  color: white;
  font-size: 1.2em;
}

div.img-controls .sliders {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  gap: 10px;
}

.rotate {
  transform: rotate(90deg);
}
.img-control-button {
  border: none;
}
div.header {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
}
.closeButton {
  color: white;
  background-color: hsla(0, 0%, 0%, 0);
  border: none;
  margin-bottom: 50px;
}

.closeButton:hover {
  cursor: pointer;
}
div.lightbox {
  width: 100vw;
  min-height: 100vh;
  background: hsla(0, 0%, 0%, 0.9);
  position: absolute;
  left: -2rem;
  top: -2rem;
  padding: 2em;
  backdrop-filter: blur(10px);
}

img,
canvas {
  max-height: 70vh;
  max-width: 90vw;
}

h1 {
  color: white;
}

.hidden {
  display: none;
}
@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
