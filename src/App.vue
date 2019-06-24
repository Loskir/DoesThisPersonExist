<template>
  <div id="app">
    <input type="file" ref="file" @change="handleFileChange">
    <canvas width="256" height="256" ref="canvas"/>
    <span class="result">It's {{result}}</span>
  </div>
</template>

<script>
  import KerasJS from 'keras-js'

  const SIZE = 256

  export default {
    name: 'app',
    data() {
      return {
        result: '',

        model: new KerasJS.Model({
          filepath: 'FM2.bin',
          gpu: true,
        }),
        modelReady: false,
      }
    },
    methods: {
      async handleFileChange(e) {
        if (!this.modelReady) {
          return
        }

        this.result = ''

        const file = e.target.files[0]

        if (!file) {
          return
        }

        const input_3 = await this.drawFile(file)

        const result = (await this.model.predict({input_3})).dense_5
        //magic with numbers

        this.result = result[0] > result[1] ? 'Fake' : 'Real'
      },

      async drawFile(file) {
        const img = new Image()

        await new Promise((res) => {
          img.onload = res
          img.src = URL.createObjectURL(file)
        })

        const canvas = this.$refs.canvas
        const context = canvas.getContext('2d')

        const w = img.width
        const h = img.height

        const sSize = Math.min(w, h)

        const sx = (w - sSize) / 2
        const sy = (h - sSize) / 2

        context.drawImage(img, sx, sy, sSize, sSize, 0, 0, SIZE, SIZE)

        const pixels = context.getImageData(0, 0, SIZE, SIZE)

        return new Float32Array(pixels.data)
          .filter((v, i) => i % 4 !== 3)
          .map(v => v / 255 * 2 - 1)
        // [-1, 1] without alpha
      },
    },
    created() {
    },
    async mounted() {
      this.model.ready().then(() => {
        this.modelReady = true
      })
    },
    beforeDestroy() {
    },
  }
</script>
<style lang="stylus">
  body
    margin 0

  #app
    font-family 'Avenir', Helvetica, Arial, sans-serif
    -webkit-font-smoothing antialiased
    -moz-osx-font-smoothing grayscale
    text-align center
    color #2c3e50
    min-height 100vh
    display flex
    flex-direction column
    justify-content center
    align-items center
</style>
