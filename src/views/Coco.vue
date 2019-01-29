<template>
  <div class="coco">
    <span v-text="status"></span>
    <video class="size" autoplay playsinline muted ref="video" width="500" height="500"/>
    <canvas @click="capture" class="size" ref="canvas" width="500" height="500"/>
  </div>
</template>

<script>
// @ is an alias to /src
import * as cocoSsd from '@tensorflow-models/coco-ssd'
import '@tensorflow/tfjs'

export default {
  name: 'coco',
  data () {
    return {
      video: {},
      stream: {},
      canvas: {},
      status: 'Загрузка...',
      frames: 0,
      captures: []
    }
  },
  mounted () {
    this.video = this.$refs.video
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      const webCamPromise = navigator.mediaDevices
        .getUserMedia({
          audio: false,
          video: {
            facingMode: 'environment'
          }
        })
        .then(stream => {
          this.stream = stream
          this.$refs.video.srcObject = stream
          return new Promise((resolve, reject) => {
            this.$refs.video.onloadedmetadata = () => {
              resolve()
            }
          })
        })
        .catch(error => {
          console.error(error)
          alert(JSON.stringify(error))
        })
      const modelPromise = cocoSsd.load()
      this.status = 'Загрузка модели...'
      Promise.all([modelPromise, webCamPromise])
        .then(values => {
          this.status = 'Модель загружена'
          this.detectFrame(this.$refs.video, values[0])
        })
        .catch(error => {
          this.status = 'Ошибка: ' + JSON.stringify(error)
          console.error(error)
          alert(JSON.stringify(error))
        })
    } else {
      alert('navigator.mediaDevices not supported')
    }
  },
  methods: {

    capture () {
      alert(1)
      this.canvas = this.$refs.canvas
      var context = this.canvas.getContext('2d')
      context.drawImage(this.video, 0, 0, 640, 480)
      this.captures.push(this.canvas.toDataURL('image/png'))
    },

    detectFrame (video, model) {
      this.frames++
      model.detect(video)
        .then(predictions => {
          this.renderPredictions(predictions)
          requestAnimationFrame(() => {
            this.detectFrame(video, model)
          })
        })
        .catch(error => {
          console.error(error)
          alert(JSON.stringify(error))
        })
    },

    renderPredictions (predictions) {
      this.canvas = this.$refs.canvas
      var context = this.canvas.getContext('2d')
      context.drawImage(this.video, 0, 0, 640, 480)
      this.captures.push(this.canvas.toDataURL('image/png'))

      // const ctx = this.$refs.canvas.getContext('2d')
      // ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height)
      // ctx.fillText(JSON.stringify(predictions), 10, 10)
      // // Font options.
      // const font = '16px sans-serif'
      // ctx.font = font
      // ctx.textBaseline = 'top'
      // predictions.forEach(prediction => {
      //   const x = prediction.bbox[0]
      //   const y = prediction.bbox[1]
      //   const width = prediction.bbox[2]
      //   const height = prediction.bbox[3]
      //   // Draw the bounding box.
      //   ctx.strokeStyle = '#00FFFF'
      //   ctx.lineWidth = 4
      //   ctx.strokeRect(x, y, width, height)
      //   // Draw the label background.
      //   ctx.fillStyle = '#00FFFF'
      //   const textWidth = ctx.measureText(prediction.class).width
      //   const textHeight = parseInt(font, 10) // base 10
      //   ctx.fillRect(x, y, textWidth + 4, textHeight + 4)
      // })

      // predictions.forEach(prediction => {
      //   const x = prediction.bbox[0]
      //   const y = prediction.bbox[1]
      //   // Draw the text last to ensure it's on top.
      //   ctx.fillStyle = '#000000'
      //   ctx.fillText(prediction.class, x, y)
      // })
    }

  }
}
</script>

<style scoped>
.size {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
