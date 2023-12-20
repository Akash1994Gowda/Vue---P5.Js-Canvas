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
      points: [],
      firstTime: true
    }
  },
  mounted() {
    let p5Handler = (p5) => {
      p5.preload= () => {
        this.imageLoad = p5.loadImage('image.png', (img) => {
          p5.resizeCanvas(img.width, img.height)
        })
      }
      p5.setup = () => {
        const canvas = p5.createCanvas(1500, 700)
        canvas.parent(this.$refs.p5Container)
        p5.imageMode(p5.CENTER)
        p5.noStroke()
      }
      p5.draw = () => {
        this.draw(p5)
      }
      p5.mouseClicked = (event) => {
        ;(this.x_value = event.clientX), (this.y_value = event.clientY)
        // if (this.firstTime) {
        this.mx = (event.clientX  - p5.width / 2) / this.zoom
        this.my = (event.clientY  - p5.height / 2) / this.zoom
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
  },
  methods: {
    draw(p5) {
      p5.background(255);
      p5.translate(p5.width / 2, p5.height / 2);
      p5.scale(this.zoom);
      p5.image(this.imageLoad, 0, 0);

      this.points.forEach(({ x, y, size }) => {
        p5.fill(255, 255, 255)
        p5.rect(x, y, size)
      })

      if (this.menuSelection === 'drawPencil') {
        p5.translate(this.mx, this.my)
        p5.translate(-this.mx, -this.my)
        p5.translate()

        p5.cursor(p5.CROSS)
        p5.strokeWeight(0.3 / this.zoom)
        p5.noFill()
        p5.stroke(69, 69, 69)
        p5.rect((p5.mouseX - p5.width / 2) / this.zoom, (p5.mouseY - p5.height / 2) / this.zoom, 1)
      }
    },
    mousePressed(mouseX, mouseY) {
      const cMouseX = (mouseX - this.p5.width / 2) / this.zoom
      const cMouseY = (mouseY - this.p5.height / 2) / this.zoom
      this.points.push({ x: cMouseX, y: cMouseY, size: 1 })
    },
    mouseWheel(event) {
      /* const delta = this.p5.constrain(event.deltaY * 0.01, -0.2, 0.2)
      this.zoom += delta
      this.zoom = this.p5.constrain(this.zoom, 0.1, 5) */

      if (this.mousePressedFlag) {
        if (event.deltaY > 0) {
          this.zoomOut()
        } else {
          this.zoomIn()
        }
      }
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
      // Sort the array of zoom scales in ascending order
      this.arrZoomScale.sort((a, b) => a - b);

      // Find the index of the zoom level in the array that is just smaller than the current zoom
      const index = this.arrZoomScale.findIndex(scale => scale >= this.zoom) - 1;

      if (index >= 0) {
        const newZoom = this.arrZoomScale[index];
        console.log(newZoom)
        this.setZoom(newZoom, 'zoomOut');
      }
    },
    setZoom(newZoom, val) {
      if (val === 'zoomIn') {
        this.sf = newZoom
      } else if (val === 'zoomOut') {
        this.sf = newZoom
      }
      this.zoom = newZoom
    },
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
