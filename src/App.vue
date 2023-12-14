<template>
  <header>
    <div class="center" id="p5Container" ref="p5Container"></div>
    <hr />
    <div class="flex">
      <h3>Mouse X - {{ x_value }}</h3>
      <h3>Mouse Y - {{ y_value }}</h3>
      <h3>Zoom - {{ sf }}</h3>
      <button @click="menuSelection = 'drawPencil'">Pencil Draw</button>
    </div>
  </header>
</template>
<script>
import p5 from 'p5'

export default {
  data() {
    return {
      imageLoad: null,
      menuSelection: null,
      mapImage: null,
      x_value: 0,
      y_value: 0,
      mousePressedFlag: false,
      sf: 1,
      mx: 0,
      my: 0,
      zoom: 1,
      loadImageWidth: 1500,
      loadImageHeight: 700,
      arrZoomScale: [
        0.1, 0.25, 0.33, 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2, 2.25, 2.5, 3, 3.5, 4, 4.5, 5, 6, 7, 8,
        10, 12
      ],
      viewPort: {
        width: 0,
        height: 0
      },
      firstTime: true
    }
  },
  mounted() {
    let p5Handler = (p5) => {
      p5.setup = () => {
        let container = this.$refs.p5Container
        let canvas = p5.createCanvas(1500, 700)
        p5.rectMode(this.p5.CENTER)
        p5.noSmooth()
        canvas.parent(container)
        this.imageLoad = p5.loadImage('https://www.google.com/imgres?imgurl=https%3A%2F%2Fi.stack.imgur.com%2Fs2hFt.png&tbnid=cY4dsfIDWC_HEM&vet=12ahUKEwjE6e-y1o6DAxXgsVYBHcjjAxUQMygAegQIARBT..i&imgrefurl=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F65808549%2Furl-has-been-blocked-by-cors-policy&docid=tSMeo9hXkzfh4M&w=1871&h=404&q=image%20link%20not%20blocked%20by%20cors%20policy&ved=2ahUKEwjE6e-y1o6DAxXgsVYBHcjjAxUQMygAegQIARBT', (img) => {
          this.mapImage = img
        })
      }
      p5.draw = () => {
        this.draw(p5)
      }
      p5.mousePressed = (event) => {
        ;(this.x_value = event.clientX), (this.y_value = event.clientY)
        // if (this.firstTime) {
        this.mx = event.clientX
        this.my = event.clientY
        this.firstTime = false
        // }

        this.mousePressedFlag = true
        this.mousePressed(event.clientX, event.clientY)
      }

      p5.mouseWheel = (event) => {
        this.mouseWheel(event)
      }
    }
    let P5 = p5
    this.p5 = new P5(p5Handler)
    this.recalViewPort()
  },
  methods: {
    draw(p5) {
      let ctx = p5
      if (this.mousePressedFlag) {
        p5.translate(this.mx, this.my)
        p5.scale(this.sf)
        p5.translate(-this.mx, -this.my)
        p5.translate()
      }

      this.drawMapImage(ctx) //ctx as arg
      if (this.menuSelection === 'drawPencil') {
        this.p5.cursor(this.p5.CROSS)
        p5.strokeWeight(0.3 / this.zoom)
        p5.noFill()
        p5.stroke(69, 69, 69)
        p5.rect(p5.mouseX, p5.mouseY, 1, 1)
      }
    },
    mousePressed(mouseX, mouseY) {
      if (this.menuSelection === 'drawPencil') {
        this.mapImage.loadPixels()
        let size = 1
        let y = mouseY
        for (let i = mouseX - size / 2; i < mouseX + size / 2; i++) {
          for (let j = y - size / 2; j < y + size / 2; j++) {
            this.mapImage.set(i, j, this.p5.color(255))
            console.log('i,j', i, j)
          }
        }
        this.mapImage.updatePixels()
        return
      }
    },
    mouseWheel(event) {
      if (this.mousePressedFlag) {
        if (event.deltaY > 0) {
          this.zoomOut()
        } else {
          this.zoomIn()
        }
      }
    },
    drawMapImage(p5) {
      let ctx = this.mapImage.drawingContext
      ctx.imageSmoothingEnabled = false
      this.imageContext = ctx

      ctx = p5.canvas.getContext('2d')
      ctx.imageSmoothingEnabled = false
      this.htmlContext = ctx

      p5.push()

      p5.image(
        this.mapImage,
        0,
        0,
        this.viewPort.width / this.zoom,
        this.viewPort.height / this.zoom
      )
      p5.pop()
    },
    zoomIn() {
      for (let zoom of this.arrZoomScale) {
        if (zoom > this.zoom) {
          this.setZoom(zoom, 'zoomIn')
          break
        }
      }
    },
    zoomOut() {
      for (let i = this.arrZoomScale.length - 1; i >= 0; i--) {
        let zoom = this.arrZoomScale[i]
        if (zoom < this.zoom) {
          this.setZoom(zoom, 'zoomOut')
          break
        }
      }
    },
    setZoom(newZoom, val) {
      if (val === 'zoomIn') {
        this.sf = newZoom
      } else if (val === 'zoomOut') {
        this.sf = newZoom
      }
      this.zoom = newZoom

      this.recalViewPort()
    },

    recalViewPort() {
      this.viewPort.width = this.zoom * this.loadImageWidth
      this.viewPort.height = this.zoom * this.loadImageHeight
    }
  }
}
</script>
<style>
#p5Container {
  height: 700px;
  width: 1500px;
}

html,
body {
  margin: 0 !important;
  padding: 0 !important;
}

.flex {
  display: flex;
  gap: 10px;
}
</style>
