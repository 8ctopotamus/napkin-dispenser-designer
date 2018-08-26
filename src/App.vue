<template>
  <div id="app">
    <link rel="stylesheet" :href="`https://fonts.googleapis.com/css?family=${fontFamily}`">

    <div class="grid-container">
      <div class="toolbar-column">
        <h3>Add Text</h3>
        <label for="text">Text</label>
        <input type="text" name="text" v-model="text" />

        <label for="font-family">Font family</label>
        <select v-model="fontFamily" name="font-family">
          <option v-for="family in fontFamilies" :key="family">{{family}}</option>
        </select>

        <label for="font-size">Font size</label>
        <input type="number" name="font-size" v-model="fontSize">
        <input type="range" name="font-size" v-model="fontSize">

        <button @click="addText" type="button" name="add-text">Add text</button>

        <h3>Add Image</h3>
        <file-upload @fileChanged="imageObj = $event" ref="fileUpload"></file-upload>
        <button type="button" name="Add image" @click="addImage">Add image</button>
      </div>
      <div class="canvas-column">
        <canvas ref="c1" id="c1" width="576" height="384"></canvas>
      </div>
    </div>

  </div>
</template>

<script>
/* eslint-disable */

import { fabric } from 'fabric'
import FileUpload from './components/FileUpload.vue'

export default {
  name: 'app',
  components: {
    FileUpload,
  },
  data () {
    return {
      text: 'Your text',
      fontFamily: 'Lato',
      fontFamilies: ['Pacifico', 'Lato', 'Sarina'],
      fontSize: 16,
      imageObj: ''
    }
  },
  mounted () {
    const canvas = new fabric.Canvas('c1', { backgroundColor: "white" })
    this.canvas = canvas
    this.canvas.setOverlayImage(require('./assets/art-area.png'), this.canvas.renderAll.bind(this.canvas))
    // window.addEventListener('mousemove',this.removeObject);
  },
  destroyed: function() {
    // window.removeEventListener('mousemove', this.removeObject);
  },
  methods: {
    addImage () {
      new fabric.Image.fromURL(this.imageObj, (img) => {
        this.canvas.add(img)
      })
      this.$refs.fileUpload.clearImageData()
    },
    addText () {
      this.canvas.add(new fabric.IText(this.text, {
        fontFamily: this.fontFamily,
        fontSize: this.fontSize,
        left: 100,
        top: 100
      }))
    },
    removeObject() {
      this.canvas.remove(this.canvas.getActiveObject())
    }
  }
}
</script>

<style>
#app {
  display: block;
  margin: 60px auto;
  max-width: 800px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.grid-container {
  display: grid;
  grid-template-columns: 1fr 576px;
  grid-column-gap: 30px;
}

.toolbar-column {
  display: flex;
  flex-direction: column;
}

canvas#c1 {
  box-shadow: 0px 0px 6px rgba(0,0,0,.35);
}
button {
  background: #006245;
  color: #fff;
}
</style>
