<template>
  <div id="app">
    <div class="fabric">
      <div class="menu">
        <h4>Menu</h4>
        <div class="field-group">
          <p class="label">Text</p>
          <textarea v-model="newText" @keyup.enter="addText" @click="setupDefaults" type="text" class="input text" rows="3" />
          <button @click="addText">Add text</button>
        </div>
        <div v-show="showEdit" class="edit">
          <div class="field-group">
            <p class="label">Font size (px)</p>
            <input v-model="fontSize" @input="changeFontSize" type="number" class="input">
          </div>
          <div class="field-group">
            <p class="label">Line height</p>
            <input v-model="lineHeight" @input="changeLineHeight" type="number" step="0.01" class="input">
          </div>
          <div class="field-group">
            <p class="label">Font family</p>
            <select class="select" @change="changeFontFamily" v-model="fontFamily">
              <option v-for="(font, index) in fonts" :key="index" :value="font"
              :style="{fontFamily: font}">{{ font }}</option>
            </select>
          </div>
        </div>
      </div>
      <div class="preview">
        <h4>Preview</h4>
        <div class="canvas-container" ref="canCont" @click="setupCurrentObject">
          <canvas ref="can"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { fabric } from 'fabric'

export default {
  name: 'App',
  data() {
    return {
      canvasWidth: null,
      canvasHeight: null,
      showEdit: false,
      fontSize: 32,
      lineHeight: 1.16,
      fontFamily: 'Arial',
      newText: '',
      textObjectsCount: null,
      fonts: [
        'Arial',
        'Roboto',
        'Fjalla One',
        'Bungee',
        'Michroma',
        'Monoton',
        'Montserrat',
        'Reenie Beanie',
        'Pacifico',
        'Dancing Script',
      ],
      canvas: {}
    }
  },
  mounted() {
    const ref = this.$refs.can
    const canvas = new fabric.Canvas(ref)
    this.canvas = canvas
    this.canvas.setWidth(this.$refs.canCont.clientWidth)
    this.canvas.setHeight(this.$refs.canCont.clientHeight)
  },
  methods: {
    addText() {
      if (this.newText !== '') {
        var text = new fabric.Text(this.newText, { 
          left: 10, 
          top: 10,
          fontSize: this.fontSize, 
          lineHeight: this.lineHeight,
          fontFamily: this.fontFamily
        })
        this.canvas.add(text)
        this.newText = ''
        this.canvas.setActiveObject(text)
        this.showEdit = true
      }
    },
    changeFontSize() {
      if ( this.canvas.getActiveObject() !== undefined ) {
        let current = this.canvas.getActiveObject()
        current.set('fontSize', parseInt(this.fontSize))
        this.canvas.renderAll()
      }
    },
    changeLineHeight() {
      if ( this.canvas.getActiveObject() !== undefined ) {
        let current = this.canvas.getActiveObject()
        current.set('lineHeight', parseFloat(this.lineHeight))
        current.set('height', parseInt(this.fontSize * this.lineHeight))
        this.canvas.renderAll()
      }
    },
    async changeFontFamily() {
      if ( this.canvas.getActiveObject() !== undefined ) {
        let current = this.canvas.getActiveObject()
        await current.set('fontFamily', this.fontFamily)
        this.canvas.renderAll()
      }

    },
    setupCurrentObject() {
      let current = this.canvas.getActiveObject() || null
      if ( current ) {
        this.fontSize = current.fontSize
        this.lineHeight = current.lineHeight
        this.fontFamily = current.fontFamily
        this.showEdit = true
        this.canvas.renderAll()
      } else {
        this.showEdit = false
      }
    },
    setupDefaults() {
      this.showEdit = false
      this.fontSize = 32
      this.lineHeight = 1.16
      this.fontFamily = 'Arial'
      this.canvas.discardActiveObject()
      this.canvas.renderAll()
    }
  } 
}
</script>

<style lang="scss">
html, body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: 'Roboto', Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: rgb(39,57,86);
  .fabric {
    width: 960px;
    height: 600px;
    box-sizing: border-box;
    background: rgb(247,248,250);
    border-radius: 5px;
    box-shadow: 0px 0px 2px 1px rgba(59, 47, 47, 0.1);
    padding: 15px 30px 30px;
    display: grid;
    grid-template-columns: 1fr 3fr;
    column-gap: 30px;
    .menu {
      display: flex;
      flex-direction: column;
      .edit {
        margin-top: 40px;
      }
      .field-group {
        margin-bottom: 20px;
        display: flex;
        flex-direction: column;
        .label {
          font-size: 0.875rem;
          margin-bottom: 4px;
          &:first-child {
            margin-top: 0;
          }
        }
        .input, select {
          border: 1px solid rgba(0,0,0,0.2);
          border-radius: 3px;
          padding: 5px 10px;
          transition: all 0.3s;
          &:focus {
            outline: 1px solid rgb(39,57,86);
          }
        }
        textarea.input {
          font-size: 1rem;
          font-family: Arial;
        }
        select {
          padding: 5px 6px;
        }
        button {
          margin-top: 10px;
          border: none;
          padding: 10px;
          background: mediumaquamarine;
          color: #FFFFFF;
          cursor: pointer;
        }
      }
    }
    .preview {
      display: flex;
      flex-direction: column;
      .canvas-container {
        background: #FFFFFF;
        border: 0.5px solid rgba(0,0,0,0.1);
        flex-grow: 1;
        box-sizing: border-box;
      }
    }
  }
}
</style>
