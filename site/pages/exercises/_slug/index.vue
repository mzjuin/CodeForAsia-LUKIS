<template>
  <div class="lol">
    <h1>Please draw a {{ actual }}</h1>
    <canvas
      id="canvas"
      v-touch:start="handleMouseDown"
      v-touch:end="handleMouseUp"
      v-touch:moving="handleMouseMove"
      width="400px"
      height="400px"
      @mousedown="handleMouseDown"
      @mouseup="handleMouseUp"
      @mousemove="handleMouseMove"
    ></canvas>
    <button type="button" class="btn btn-primary" @click="predict">
      Predict
    </button>
    <button type="button" class="btn btn-danger" @click="() => {}">
      Report
    </button>
    <p>Score {{ totalCorrect }} / {{ totalAnswer }}</p>
  </div>
</template>

<script>
import Vue from 'vue'
// import Tesseract from 'tesseract.js'
import Vue2TouchEvents from 'vue2-touch-events'
Vue.use(Vue2TouchEvents)

export default {
  data() {
    return {
      mouse: {
        current: {
          x: 0,
          y: 0
        },
        previous: {
          x: 0,
          y: 0
        },
        down: false
      },
      // letters: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ',
      letters: 'ABCDEFGHJKLMNOPRSTUVWYZ',
      actual: 'A',
      totalAnswer: 0,
      totalCorrect: 0,
      answered: {}
    }
  },
  computed: {
    currentMouse: function() {
      const c = document.getElementById('canvas')
      const rect = c.getBoundingClientRect()

      return {
        x: this.mouse.current.x - rect.left,
        y: this.mouse.current.y - rect.top
      }
    }
  },
  asyncData({
    isDev,
    route,
    store,
    env,
    params,
    query,
    req,
    res,
    redirect,
    error
  }) {
    return {
      exerciseName: params.slug
    }
  },
  mounted() {
    this.actual = this.randomChoice()
  },
  methods: {
    resetQuestion() {
      this.actual = this.randomChoice()
      const c = document.getElementById('canvas')
      const ctx = c.getContext('2d')
      // Store the current transformation matrix
      // ctx.save()

      // Use the identity matrix while clearing the canvas
      ctx.clearRect(0, 0, c.width, c.height)
      const w = c.width
      c.width = 1
      c.width = w
      // Restore the transform
      // ctx.restore()
    },
    randomChoice: function() {
      const arr = this.letters
      return arr[Math.floor(arr.length * Math.random())]
    },
    test: function(params) {
      debugger
    },
    predict: async function(lala) {
      this.totalAnswer += 1
      const c = document.getElementById('canvas')
      // const ctx = c.getContext('2d')
      // const data = await Tesseract.recognize(ctx)
      const dataURL = c.toDataURL()
      console.log(dataURL)
      const opts = {
        data_to_predict: dataURL
      }
      const data = await (await fetch(
        'http://localhost:5000/bijaksana/predict',
        {
          method: 'post',
          body: JSON.stringify(opts)
        }
      )).json()
      if (data.result.includes(this.actual)) {
        alert('Correct')
        this.totalCorrect += 1
        this.answered[this.actual] += 1
      } else {
        alert('You draw ' + data.result + ' but expected ' + this.actual)
      }
      this.resetQuestion()
      console.log(data)
    },
    draw: function(event) {
      // requestAnimationFrame(this.draw);
      // console.log('-----------------inside draw------------------')
      // console.log('currentMouse', this.currentMouse, this.mouse.down)
      if (this.mouse.down) {
        const c = document.getElementById('canvas')

        const ctx = c.getContext('2d')

        ctx.clearRect(0, 0, 400, 400)

        ctx.lineTo(this.currentMouse.x, this.currentMouse.y)
        ctx.strokeStyle = '#ffffff'
        ctx.fillStyle = 'black'

        ctx.fillRect(0, 0, c.width, c.height)
        ctx.lineWidth = 8
        ctx.stroke()
      }
    },
    handleMouseDown: function(event) {
      // console.log('---------mouse down-----------')
      // console.log('pageXY', event.pageX, event.pageY)
      // console.log('currentMouse', this.currentMouse.x, this.currentMouse.y)
      this.mouse.down = true
      this.mouse.current = {
        x: event.pageX || event.touches[0].pageX,
        y: event.pageY || event.touches[0].pageY
      }

      const c = document.getElementById('canvas')
      const ctx = c.getContext('2d')
      ctx.moveTo(this.currentMouse.x, this.currentMouse.y)
    },
    handleMouseUp: function() {
      // console.log('mouse up')
      this.mouse.down = false
    },
    handleMouseMove: function(event) {
      // console.log('mouse move')
      this.mouse.current = {
        x: event.pageX || event.touches[0].pageX,
        y: event.pageY || event.touches[0].pageY
      }

      this.draw(event)
    }
  }
}
</script>

<style>
body {
  background: #eee;
}

canvas {
  box-shadow: 0px 2px 3px rgba(0, 0, 0, 0.2);
  border: 1px solid black;
}

.lol {
  margin-top: -56px;
  padding: 100px 50px 20px 50px;
  background-image: url('/paster4.jpg');
  background-size: 100% 100%;
  background-repeat: no-repeat;
  height: -webkit-fill-available;
}
</style>
