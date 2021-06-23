<template>
  <div id="app">
    <div class="navbar">
      <h2 class="app-title">Text Editor</h2>
    </div>
    <main class="content">
      <div class="app-container">
        <div class="preview">
          <div class="canvas-container" ref="canCont">
            <canvas ref="can"></canvas>
          </div>
        </div>
      </div>
      <div class="sidebar">
        <h4 class="sidebar-title">Text Layer</h4>
        <div class="field-group">
          <p class="label">Enter Text</p>
          <div class="text-group">
            <input v-model="newText" @keyup.enter="addText" @click="setupDefaults" type="text" class="input"/>
            <button @click="addText">+</button>
          </div>
        </div>
        <div v-show="showEdit" class="edit">
          <div class="sizes">
            <div class="size-group">
              <p class="label">Font size</p>
              <input v-model="fontSize" @input="changeFontSize" type="number" class="input">
              <span class="px"
              :style="{ left: (fontSize.toString().length * 9) + 14 +'px' }">px</span>
            </div>
            <div class="size-group">
              <p class="label">Line height</p>
              <input v-model="lineHeight" @input="changeLineHeight" type="number" step="0.01" class="input">
            </div>
          </div>
          <div class="field-group">
            <p class="label">Change font family</p>
            <select class="input" @change="changeFontFamily" v-model="fontFamily">
              <option v-for="(font, index) in fonts" :key="index" :value="font"
              :style="{fontFamily: font}">{{ font }}</option>
            </select>
          </div>
        </div>
        <div v-show="showEdit" class="info">
          <ul>
            <li>Font size: {{ fontSize }}</li>
            <li>Line height: {{ lineHeight }}</li>
            <li v-if="currentObjectScaleY">ScaleY: {{ currentObjectScaleY }}</li>
            <li v-if="currentObjectScaleX">ScaleX: {{ currentObjectScaleX }}</li>
          </ul>
        </div>
      </div>
    </main>
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
      newText: '',
      showEdit: false,
      fontSize: 32,
      lineHeight: 1.5,
      currentObjectScaleY: null,
      currentObjectScaleX: null,
      fontFamily: 'Roboto',
      fonts: [
        'Arial',
        'Roboto',
        'Fjalla One',
        'Staatliches',
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

    this.canvas.on('object:modified', (e) => {
      console.log(e.target)
      this.currentObjectScaleY = e.target.scaleY
      this.currentObjectScaleX = e.target.scaleX
      // this.setupResizedValues()
    })

    this.canvas.on('mouse:up', (e) => {
      console.log(e.target)
      this.setupCurrentObject()
    })
  },
  methods: {
    addText() {
      if (this.newText !== '') {
        var text = new fabric.IText(this.newText, { 
          left: 10, 
          top: 10,
          fontSize: this.fontSize, 
          lineHeight: this.lineHeight,
          fontFamily: this.fontFamily,
          fill: '#424242'
        })
        this.canvas.add(text)
        this.canvas.setActiveObject(text)
        this.showEdit = true
      }
    },
    changeFontSize() {
      if ( this.canvas.getActiveObject() !== undefined ) {
        let current = this.canvas.getActiveObject()
        current.set('fontSize', parseInt(this.fontSize))
        current.set('height', parseInt(this.fontSize * this.lineHeight))
        this.canvas.renderAll()
      }
    },
    changeLineHeight() {
      if ( this.canvas.getActiveObject() !== undefined ) {
        let current = this.canvas.getActiveObject()
        current.set('lineHeight', parseFloat(this.lineHeight))
        // current.set('height', parseInt(this.fontSize * this.lineHeight))
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
    // setupResizedValues() {
    //     let current = this.canvas.getActiveObject()
    //     let newFontSize = Math.round(current.scaleY * this.fontSize)
    //     // let newLineHeight = Math.round(current.scaleY * this.lineHeight)
    //     current.set('fontSize', parseInt(newFontSize))
    //     current.set('scaleY', 1)
    //     current.set('scaleX', current.scaleX)
    // },
    setupCurrentObject() {
      let current = this.canvas.getActiveObject() || null
      if ( current ) {
        this.newText = current.text
        this.fontSize = current.fontSize
        this.lineHeight = current.lineHeight
        this.fontFamily = current.fontFamily
        this.currentObjectScaleY = current.scaleY
        this.currentObjectScaleX = current.scaleX
        this.showEdit = true
        this.canvas.renderAll()
      } else {
        this.showEdit = false
        this.newText = ''
        this.currentObjectScaleY = null
        this.currentObjectScaleX = null
      }
    },
    setupDefaults() {
      this.showEdit = false
      this.fontSize = 32
      this.lineHeight = 1.16
      this.fontFamily = 'Roboto'
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
  height: 100vh;
}

#app {
  font-family: 'Roboto', Arial, Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  .navbar {
    width: 100%;
    min-height: 80px;
    background: #43ce93;
    display: flex;
    justify-content: center;
    align-items: center;
    .app-title {
      margin: 0;
      color: #FFFFFF;
    }
  }
  .content {
    max-width: 100%;
    display: flex;
    height: calc(100vh - 80px);
    min-height: calc(100vh - 80px);
    .app-container {
      flex-grow: 1;
      background: #e5e5e5;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      .preview {
        width: 664px;
        height: 664px;
        border: 1px solid #979797;
        background: #FFFFFF;
        padding: 10px;
        box-sizing: border-box;
        .canvas-container {
          background: #FFFFFF;
          height: 100%;
          box-sizing: border-box;
        }
      }
    }
    .sidebar {
      width: 368px;
      flex-grow: 0;
      display: flex;
      flex-direction: column;
      box-sizing: border-box;
      box-shadow: -7px 0px 13px 1px #c0c0c0;
      padding: 2.25rem;
      .sidebar-title {
        color: #888888;
        font-size: 1.25rem;
        margin-top: 0;
        border-bottom: 1px solid #d1d1d1;
        padding-bottom: 1rem;
      }
      .label {
        margin: 0 0 0.5rem;
        color: #666666;
        font-size: 1rem;
      }
      .field-group {
        margin-bottom: 30px;
        display: flex;
        flex-direction: column;
        .input {
          border: 1px solid #cfcfcf;
          color: #6b6b6b;
          font-size: 1rem;
          border-radius: 5px;
          padding: 0 0 0 7px;
          height: 30px;
        }
        select {
          cursor: pointer;
          padding-left: 6px;
          option {
            font-family: 'Roboto', Arial;
          }
          
        }
        .text-group {
          display: flex;
          align-items: stretch;
          input {
            flex-grow: 1;
          }
          button {
            margin-left: 20px;
            padding-top: -2px;
            width: 30px;
            border-radius: 5px;
            box-shadow: none;
            border: none;
            color: #FFFFFF;
            font-weight: bold;
            font-size: 1.5rem;
            background: #43ce93;
            cursor: pointer;
            transition: all 0.2s;
            &:hover {
              background: lighten(#43ce93, 10%);
            }
          }
        }
      }
      .sizes {
        width: 100%;
        max-width: 100%;
        display: grid;
        grid-template-columns: 1fr 1fr;
        column-gap: 20px;
        .size-group {
          flex-grow: 0;
          box-sizing: border-box;
          display: flex;
          flex-direction: column;
          margin-bottom: 30px;
          position: relative;
          .input {
            width: 100%;
            max-width: 100%;
            box-sizing: border-box;
            border: 1px solid #cfcfcf;
            color: #6b6b6b;
            font-size: 1rem;
            border-radius: 5px;
            padding: 0 0 0 7px;
            height: 30px;
          }
          .px {
            position: absolute;
            top: 32px;
            color: #6b6b6b;
          }
        }
      }
      .info {
        ul {
          list-style: none;
          padding-left: 0;
          color: #6b6b6b;
          font-size: 0.875rem;
        }
      }
    }
  }
}
</style>
