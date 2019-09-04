<template>
  <div>
    <div class="row">
      <div class="col" style="margin-left:100px;">
        <h4 id="status">Loading Model...</h4>
        <canvas
          id="canvas"
          width="300"
          height="300"
          class="canvas"
          style="border:1px solid #b9bfc9;margin-top:25px;"
        ></canvas>
        <div class="btn-group" style="margin-top:40px; ">
          <input
            id="myRange"
            type="range"
            min="5"
            max="20"
            value="10"
            class="slider"
            style="margin-top:20px;"
          />
          <button
            type="button"
            class="btn btn-outline-primary"
            onclick="erase()"
            style="margin-left:10px;"
            disabled
          >
            Clear
          </button>
        </div>
      </div>
      <div class="col">
        <section style="margin-top:120px">
          <div class="pieID pie"></div>
          <ul class="pieID legend">
            <li>
              <em id="sym1"></em>
              <span id="prob1"></span>
            </li>
            <li>
              <em id="sym2"></em>
              <span id="prob2"></span>
            </li>
            <li>
              <em id="sym3"></em>
              <span id="prob3"></span>
            </li>
            <li>
              <em id="sym4"></em>
              <span id="prob4"></span>
            </li>
            <li>
              <em id="sym5"></em>
              <span id="prob5"></span>
            </li>
          </ul>
        </section>
      </div>
    </div>
    <p id="txt"></p>
    <script src="pie.js"></script>
  </div>
</template>

<script>
window.$ = require('jquery')
export default {
  mounted() {
    function setTable(top5, probs) {
      for (let i = 0; i < top5.length; i++) {
        const sym = document.getElementById('sym' + (i + 1))
        const prob = document.getElementById('prob' + (i + 1))
        sym.innerHTML = top5[i]
        prob.innerHTML = Math.round(probs[i] * 100)
      }
      createPie('.pieID.legend', '.pieID.pie')
    }

    // prepare the drawing canvas
    $(function() {
      canvas = window._canvas = new fabric.Canvas('canvas')
      canvas.backgroundColor = '#ffffff'
      canvas.isDrawingMode = 0
      canvas.freeDrawingBrush.color = 'black'
      canvas.freeDrawingBrush.width = 10
      canvas.renderAll()
      canvas.on('mouse:up', function(e) {
        getFrame()
        mousePressed = false
      })
      canvas.on('mouse:down', function(e) {
        mousePressed = true
      })
      canvas.on('mouse:move', function(e) {
        recordCoor(e)
      })
    })

    // record the current drawing coordinates
    function recordCoor(event) {
      const pointer = canvas.getPointer(event.e)
      const posX = pointer.x
      const posY = pointer.y
      if (posX >= 0 && posY >= 0 && mousePressed) {
        coords.push(pointer)
      }
    }

    // get the best bounding box by trimming around the trimming
    function getMinBox() {
      const minX = 0
      const minY = 0
      const maxX = 300
      const maxY = 300

      const coorX = coords.map(function(p) {
        return p.x
      })
      const coorY = coords.map(function(p) {
        return p.y
      })

      const min_coords = {
        x: Math.min.apply(null, coorX),
        y: Math.min.apply(null, coorY)
      }
      const max_coords = {
        x: Math.max.apply(null, coorX),
        y: Math.max.apply(null, coorY)
      }
      return {
        min: min_coords,
        max: max_coords
      }
    }

    // get the current frame of the canvas
    function getFrame() {
      // make sure we have at least two recorded coordinates
      if (coords.length >= 2) {
        // get the minimum bounding box
        const mbb = getMinBox()
        const dpi = window.devicePixelRatio
        const imgData = canvas.contextContainer.getImageData(
          mbb.min.x * dpi,
          mbb.min.y * dpi,
          (mbb.max.x - mbb.min.x) * dpi,
          (mbb.max.y - mbb.min.y) * dpi
        )
        // get the predictions, top 5
        const pred = model.predict(preprocess(imgData)).dataSync()
        const indices = findIndicesOfMax(pred, 5)
        const probs = findTopValues(pred, 5)
        const symbols = getSymbols(indices)
        // set the table
        setTable(symbols, probs)
      }
    }
    // get the latex symbols by indices
    function getSymbols(indices) {
      const outp = []
      for (let i = 0; i < indices.length; i++) outp[i] = symbols[indices[i]]
      return outp
    }

    // load the class names
    async function loadDict() {
      await $.ajax({
        url: 'model2/class_names.txt',
        dataType: 'text'
      }).done(success)
    }

    // load the class names
    function success(data) {
      const lst = data.split(/\n/)
      symbols = []
      for (let i = 0; i < lst.length - 1; i++) {
        const symbol = lst[i]
        symbols[i] = symbol
      }
    }

    // get indices of the top probs
    function findIndicesOfMax(inp, count) {
      const outp = []
      for (let i = 0; i < inp.length; i++) {
        outp.push(i) // add index to output array
        if (outp.length > count) {
          outp.sort(function(a, b) {
            return inp[b] - inp[a]
          }) // descending sort the output array
          outp.pop() // remove the last index (index of smallest element in output array)
        }
      }
      return outp
    }

    // find the top 5 predictions
    function findTopValues(inp, count) {
      const outp = []
      const indices = findIndicesOfMax(inp, count)
      // show 5 greatest scores
      for (let i = 0; i < indices.length; i++) outp[i] = inp[indices[i]]
      return outp
    }

    // preprocess the data
    function preprocess(imgData) {
      return tf.tidy(() => {
        const tensor = tf.browser.fromPixels(imgData).toFloat()
        const offset = tf.scalar(255.0)
        // Normalize the image
        const normalized = tf.scalar(1.0).sub(tensor.div(offset))
        const resized = tf.image.resizeBilinear(normalized, [28, 28])
        const sliced = resized.slice([0, 0, 1], [28, 28, 1])
        const batched = sliced.expandDims(0)
        return batched
      })
    }

    // load the model
    async function loadModel() {
      model = await tf.loadLayersModel('model2/model.json')
      // warm up
      model.predict(tf.zeros([1, 28, 28, 1]))
      allowDrawing()
      await loadDict()
    }

    // allow drawing on canvas
    function allowDrawing() {
      canvas.isDrawingMode = 1
      document.getElementById('status').innerHTML = 'Model Loaded'
      $('button').prop('disabled', false)
      const slider = document.getElementById('myRange')
      slider.oninput = function() {
        canvas.freeDrawingBrush.width = this.value
      }
    }
    function erase() {
      canvas.clear()
      canvas.backgroundColor = '#ffffff'
      coords = []
    }

    loadModel()

    let model
    var canvas
    var symbols = [{}]
    var canvas
    var coords = []
    var mousePressed = false
  }
}
</script>

<style scoped>
.table tbody tr td {
  font-family: 'Courier New', Courier, 'Lucida Sans Typewriter',
    'Lucida Typewriter', monospace;
}
table thead tr th {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 24px;
  font-weight: 400;
  line-height: 26.4px;
}
.sym {
  font-size: 50px;
}
.slider {
  -webkit-appearance: none;
  width: 230px;
  height: 1px;
  border-radius: 3px;
  background: #d3d3d3;
  outline: none;
  opacity: 0.9;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: #007aff;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: #007aff;
  cursor: pointer;
}
</style>
