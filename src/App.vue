<template>
  <div id="app" class="container">
    <div v-if="showUI" class="row">
      <div class="col-sm-12 col-md-6">
        <accordion title="Background">
          <chrome-picker :value="color" @input="updateColor"></chrome-picker>
          <button type="button" name="set-background-color" @click="setCanvasBackgroundColor" class="btn btn-block">
            Set Background Color
          </button>
        </accordion>

        <accordion title="Add Text">
          <label for="text">Text</label>
          <input type="text" name="text" v-model="text" class="form-control" />
          <label for="font-family">Font family</label>
          <select v-model="fontFamily" name="font-family" class="form-control">
            <option v-for="family in fontFamilies" :key="family" :style="`font-family: ${family};`" :label="family">{{family}}</option>
          </select>
          <label for="font-size">Font size</label>
          <input type="number" name="font-size" v-model="fontSize" class="form-control">
          <input type="range" name="font-size" v-model="fontSize">
          <chrome-picker :value="color" @input="updateColor"></chrome-picker>
          <button @click="addText" type="button" name="add-text" class="btn btn-block">Add text</button>
        </accordion>

        <accordion title="Add Image">
          <file-upload @fileChanged="imageObj = $event" ref="fileUpload"></file-upload>
          <button type="button" name="Add image" @click="addImage" :disabled="imageObj === ''" class="btn btn-block">Add image</button>
        </accordion>

        <accordion title="Add Shapes" class="accordion-shapes">
          <button type="button" name="add-circle" @click="addShape('circle')" class="btn">
            <i class="fas fa-circle"></i> Add Circle
          </button>
          <br/>
          <button type="button" name="add-square" @click="addShape('rectangle')" class="btn">
            <i class="fas fa-square"></i> Add Square
          </button>
          <h5>Add Polygons and Stars</h5>
          <div class="form-group">
            <label for="corners">Number of Corners</label>
            <input type="number" name="corners" v-model="corners" class="form-control" />
            <br/>
            <button type="button" name="add-polygon" @click="addShape('polygon')" class="btn">
              Add Polygon
            </button>
            <br/>
            <button type="button" name="add-polygon" @click="addShape('star')" class="btn">
              Add Star
            </button>
          </div>
          <chrome-picker :value="color" @input="updateColor"></chrome-picker>
        </accordion>
      </div>

      <div class="col-sm-12 col-md-6">
        <canvas ref="c1" id="c1" width="442" height="298"></canvas>

        <div class="canvas-toolbar">
          <button type="button" name="bring-forward" title="Bring forward" @click="sortLayer('up')" :disabled="!isActiveObject" class="btn">
            <i class="fas fa-sort-up"></i>
          </button>
          <button type="button" name="send-backwards" title="Send backwards" @click="sortLayer('down')" :disabled="!isActiveObject" class="btn">
            <i class="fas fa-sort-down"></i>
          </button>
          <button type="button" name="delete-object" title="Delete object" @click="removeObject" :disabled="!isActiveObject" class="btn">
            <i class="fas fa-trash-alt"></i>
          </button>
          <button type="button" name="unselect-object" title="Delete object" @click="deselectObject" :disabled="!isActiveObject" class="btn">
            <i class="fas fa-hand-paper"></i> Release Object
          </button>
          <button @click="save" type="button" title="Save Design" name="save" class="btn">Save</button>
        </div>
        <h5 v-if="layers.length > 0" class="layers-title">Layers:</h5>
        <div v-for="(layer, i) in layers" :key="i" class="layer">
          {{layer.type}}
        </div>

      </div>
    </div>
    <div v-else class="row desktop-only">
      <div class="col text-center">
        <i class="fas fa-desktop fa-4x"></i>
        <p>Sorry, this app is intended for desktop use.</p>
      </div>
    </div>

  </div>
</template>

<script>
/* eslint-disable */
import { fabric } from 'fabric'
import FileUpload from './components/FileUpload.vue'
import Accordion from './components/Accordion.vue'
import { saveAs } from 'file-saver'
import canvasToBlob from 'canvas-toBlob'
import { Chrome } from 'vue-color'

function regularPolygonPoints(sideCount,radius){
  var sweep=Math.PI*2/sideCount
  var cx=radius
  var cy=radius
  var points=[]
  for(var i=0;i<sideCount;i++){
    var x=cx+radius*Math.cos(i*sweep)
    var y=cy+radius*Math.sin(i*sweep)
    points.push({x:x,y:y})
  }
  return(points)
}

function starPolygonPoints(spikeCount, outerRadius, innerRadius) {
  var rot = Math.PI / 2 * 3
  var cx = outerRadius
  var cy = outerRadius
  var sweep = Math.PI / spikeCount
  var points = []
  var angle = 0
  for (var i = 0; i < spikeCount; i++) {
    var x = cx + Math.cos(angle) * outerRadius
    var y = cy + Math.sin(angle) * outerRadius
    points.push({x: x, y: y})
    angle += sweep

    x = cx + Math.cos(angle) * innerRadius
    y = cy + Math.sin(angle) * innerRadius
    points.push({x: x, y: y})
    angle += sweep
  }
  return (points)
}

var colors = {
  hex: '#194d33',
  hsl: { h: 150, s: 0.5, l: 0.2, a: 1 },
  hsv: { h: 150, s: 0.66, v: 0.30, a: 1 },
  rgba: { r: 25, g: 77, b: 51, a: 1 },
  a: 1
}

export default {
  name: 'app',
  components: {
    Accordion,
    FileUpload,
    'chrome-picker': Chrome,
  },
  data () {
    return {
      showUI: true,
      colors,
      color: '#333333',
      text: 'Your text',
      fontFamily: 'Lato',
      fontFamilies: [
        'Abril FatFace',
        'Archivo',
        'Arvo',
        'Concert One',
        'Exo 2',
        'Lato',
        'Lena',
        'Lora',
        'Merriweather',
        'Montserrat',
        'Noto Sans',
        'Nunito',
        'Oswald',
        'Pacifico',
        'Playfair Display',
        'Raleway',
        'Roboto',
        'Sarina',
        'Spectral',
      ],
      fontSize: 16,
      imageObj: '',
      corners: 3,
      isActiveObject: false,
      layers: [],
    }
  },
  mounted () {
    this.canvas = new fabric.Canvas('c1', { backgroundColor: "white" })
    this.canvas.setOverlayImage(require('./assets/art-boundaries.png'), this.canvas.renderAll.bind(this.canvas))
    this.canvas.on('mouse:down', this.checkActiveObject)

    if (matchMedia) {
      this.mq = window.matchMedia("(min-width: 767px)")
      this.mq.addListener(this.widthChange)
      this.widthChange(this.mq)
    }
  },
  destroyed: function() {
    this.mq.removeListener(this.widthChange)
  },
  methods: {
    addImage () {
      new fabric.Image.fromURL(this.imageObj, img => {
        this.canvas.add(img)
        this.getLayers()
      })
      this.$refs.fileUpload.clearImageData()
      this.imageObj = ''
    },
    addShape (shape, corners) {
      var object
      if (shape === 'circle') {
        object = new fabric.Circle({
          radius: 30,
          fill: this.color,
          top: 100,
          left: 100
        })
      }
      if (shape === 'rectangle') {
        object = new fabric.Rect({
          left: 100,
          top: 100,
          fill: this.color,
          width: 20,
          height: 20
        })
      }
      if (shape === 'polygon') {
        var points = regularPolygonPoints(this.corners, 30);
        object = new fabric.Polygon(points, {
          fill: this.color,
          left: 150,
          top: 10,
        }, false);
      }
      if (shape === 'star') {
        // make a star
        var points = starPolygonPoints(this.corners,50,25);
        object = new fabric.Polygon(points, {
          fill: this.color,
          left: 150,
          top: 10,
        }, false);
      }

      this.canvas.add(object)
      this.canvas.renderAll()
      this.getLayers()
    },
    addText () {
      this.canvas.add(new fabric.IText(this.text, {
        fontFamily: this.fontFamily,
        fontSize: this.fontSize,
        fill: this.color,
        left: 100,
        top: 100
      }))

      this.getLayers()
    },
    deselectObject () {
      this.canvas.discardActiveObject()
      this.canvas.renderAll()
    },
    checkActiveObject (options) {
      if (options.target) {
        this.isActiveObject = true
      } else {
        this.isActiveObject = false
      }
    },
    getLayers () {
      this.layers = this.canvas.getObjects().reverse()
    },
    removeObject() {
      this.canvas.remove(this.canvas.getActiveObject())
      this.isActiveObject = false
    },
    save () {
      // ensure that the saved PNG is a certain size
      var resizedCanvas = document.createElement("canvas")
      var resizedContext = resizedCanvas.getContext("2d")

      // 8.5 x 11
      resizedCanvas.width = "612"
      resizedCanvas.height = "792"

      // remove overlay image for saving
      this.canvas.setOverlayImage(null, this.canvas.renderAll.bind(this.canvas))

      var canvas = document.getElementById("c1")
      var context = canvas.getContext("2d")

      resizedContext.drawImage(canvas, 86, 80, 442, 298)
      resizedContext.drawImage(canvas, 86, 414, 442, 298)

      resizedCanvas.toBlob(blob => {
        saveAs(blob, 'my-design.png')
        // reset overlay image
        this.canvas.setOverlayImage(require('./assets/art-boundaries.png'), this.canvas.renderAll.bind(this.canvas))
      })
    },
    setCanvasBackgroundColor () {
      this.canvas.setBackgroundColor(this.color)
      this.canvas.renderAll()
    },
    sortLayer(direction) {
      const ao = this.canvas.getActiveObject()
      if (ao) {
        if (direction === 'up') {
          this.canvas.bringForward(ao)
        }
        if (direction === 'down') {
          this.canvas.sendBackwards(ao)
        }
        this.getLayers()
        this.canvas.renderAll()
      }
    },
    updateColor (color) {
      this.color = color.hex
      if (this.canvas.getActiveObject()) {
        this.canvas.getActiveObject().setColor(this.color)
        this.canvas.renderAll()
      }
    },
    widthChange(mq) {
      if (mq.matches) {
        this.showUI = true
      } else {
        this.showUI = false
      }
    },
  }
}
</script>

<style>
#app {
  display: block;
  margin: 60px auto;
  max-width: 860px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.add-column {
  display: flex;
  flex-direction: column;
}
canvas#c1 {
  box-shadow: 0px 0px 6px rgba(0,0,0,.35);
  margin-bottom: 25px;
}
.canvas-toolbar {
  display: flex;
  margin-top: 15px;
}
.canvas-toolbar button {
  margin-right: 5px;
}
.canvas-toolbar button[name="save"] {
  margin-left: auto;
  margin-right: 0;
}
button {
  background: #006245;
  color: #fff;
  margin-bottom: 10px;
}
button:disabled {
  opacity: 1;
}
.layers-title {
  margin-top: 15px;
}
.layer {
  background: #bbb;
  border-bottom: 1px solid #999;
  padding: 5px;
}
.desktop-only i {
  color: #999;
  margin-bottom: 10px;
}
</style>
