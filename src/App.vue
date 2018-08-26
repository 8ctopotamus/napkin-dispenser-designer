<template>
  <div id="app">
    <canvas ref="c1" id="c1" width="576" height="384"></canvas>

    <input type="text" v-model="text" />
    <button @click="addText" type="button" name="add-text">Add text</button>

    <select>
      <option value="Lato">Lato</option>
      <option value="Pacifico">Pacifico</option>
    </select>

    <label for="font-size">Font size</label>
    <input type="number" name="font-size" v-model="fontSize">
    <input type="range" name="font-size" v-model="fontSize">

    <file-upload @fileChanged="imageObj = $event"></file-upload>
    <button type="button" name="Add image" @click="addImage">Add image</button>
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
      text: 'Add text',
      fontFamily: 'Lato',
      fontSize: 16,
      imageObj: ''
    }
  },
  mounted () {
    const canvas = new fabric.Canvas('c1', { backgroundColor: "white" })
    this.canvas = canvas

    canvas.selectionColor = 'rgba(0,255,0,0.3)';
    canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));
    canvas.selectionBorderColor = 'red';
    canvas.selectionLineWidth = 5;

  },
  methods: {
    addImage () {
      new fabric.Image.fromURL(this.imageObj, (img) => {
        this.canvas.add(img)
      })
    },
    addText () {
      this.canvas.add(new fabric.IText(this.text, {
        fontFamily: this.fontFamily,
        left: 100,
        top: 100
      }))
    }
  }
}
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 60px;
  margin-bottom: 60px;
}
canvas {
  box-shadow: 0px 0px 3px rgba(0,0,0,.25);
}
button {
  background: #006245;
  color: #fff;
}
</style>
