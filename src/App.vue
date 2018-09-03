<template>
  <div id="app">
    <div class="grid-container">
      <div class="toolbar-column">
        <accordion title="Canvas">
          <chrome-picker :value="color" @input="updateColor"></chrome-picker>
          <button type="button" name="set-background-color" @click="setCanvasBackgroundColor" class="btn btn-block">
            Set Background Color
          </button>
        </accordion>

        <accordion title="Text">
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

        <accordion title="Image">
          <file-upload @fileChanged="imageObj = $event" ref="fileUpload"></file-upload>
          <button type="button" name="Add image" @click="addImage" :disabled="imageObj === ''" class="btn btn-block">Add image</button>
        </accordion>
      </div>

      <div class="canvas-column">
        <canvas ref="c1" id="c1" width="576" height="384"></canvas>

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
          <button @click="save" type="button" name="save" class="btn">Save</button>
        </div>

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
      isActiveObject: false
    }
  },
  mounted () {
    this.canvas = new fabric.Canvas('c1', { backgroundColor: "white" })
    this.canvas.setOverlayImage(require('./assets/art-boundaries.png'), this.canvas.renderAll.bind(this.canvas))
    this.canvas.on('mouse:down', this.checkActiveObject)
    // window.addEventListener('keydown', this.removeObject);
  },
  destroyed: function() {
    // window.removeEventListener('keydown', this.removeObject);
  },
  methods: {
    addImage () {
      new fabric.Image.fromURL(this.imageObj, img => this.canvas.add(img))
      this.$refs.fileUpload.clearImageData()
      this.imageObj = ''
    },
    addText () {
      this.canvas.add(new fabric.IText(this.text, {
        fontFamily: this.fontFamily,
        fontSize: this.fontSize,
        fill: this.color,
        left: 100,
        top: 100
      }))
    },
    checkActiveObject (options) {
      if (options.target) {
        // console.log(options.target.type)
        this.isActiveObject = true
      } else {
        this.isActiveObject = false
      }
    },
    removeObject() {
      this.canvas.remove(this.canvas.getActiveObject())
      this.isActiveObject = false
    },
    save () {
      // this.canvas.setOverlayImage(require('./assets/art-area.png'), this.canvas.renderAll.bind(this.canvas))

      // const shape = this.canvas.overlayImage
      // console.log(shape)
      // var shape = this.canvas.getActiveObject();
      // this.canvas.remove(shape);
      // this.canvas.clipTo = function(ctx) {
      //   shape.render(ctx);
      // };

      this.$refs.c1.toBlob(blob => {
        saveAs(blob, 'my-design.png')
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
        this.canvas.renderAll()
      }
    },
    updateColor (color) {
      this.color = color.hex
      if (this.canvas.getActiveObject()) {
        this.canvas.getActiveObject().setColor(this.color)
        this.canvas.renderAll();
      }
    }
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
.grid-container {
  display: grid;
  grid-template-columns: 1fr 576px;
  grid-column-gap: 25px;
}
.toolbar-column {
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
}
button:disabled {
  opacity: .55;
}
</style>
