<template>
  <div class="image-upload">
    <div class="wrapper">
      <canvas width="300" height="300"></canvas>
      <div class="controls">
        <button class="zoom-out" @click.prevent="zoomOut">ZoomOut</button>
        <div class="slide">
          <div class="slide-track">
            <div class="slide-dot"  :style="`left:${percent}%`"></div>
          </div>
        </div>
        <button class="zoom-in" @click.prevent="zoomIn">ZoomIn</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'image-upload',
  props: ['image', 'imageData'],
  data () {
    return {
      scale: 1,
      minScale: 0.1,
      maxScale: 3,
      percent: 1 / (3 - 0.1) * 100
    }
  },
  computed: {
    imageInformation () {
      return {
        name: this.image.name
      }
    }
  },
  watch: {
    scale () {
      if (this.scale > this.maxScale) {
        this.scale = this.maxScale
      }
      if (this.scale < this.minScale) {
        this.scale = this.minScale
      }
      this.percent = (this.scale - this.minScale) / (this.maxScale - this.minScale) * 100
      this.drawImage()
    }
  },
  mounted () {
    this.render()
    this.slideZoom()
  },
  methods: {
    render () {
      let data = this.imageData
      let image = new Image()
      let containerWidth = 300
      let containerHeight = 300
      let width
      let height
      let canvas = this.$el.querySelector('canvas')
      let ctx = canvas.getContext('2d')

      let drawImage = () => {
        let dx = (containerWidth - width * this.scale) / 2
        let dy = (containerHeight - height * this.scale) / 2
        ctx.clearRect(0, 0, width * this.maxScale, height * this.maxScale)
        ctx.drawImage(image, dx, dy, width * this.scale, height * this.scale)
      }
      this.drawImage = drawImage
      image.onload = () => {
        width = image.width
        height = image.height
        drawImage(image)
      }
      image.src = data
    },
    zoomIn () {
      this.scale += 0.1
    },
    zoomOut () {
      this.scale -= 0.1
    },
    slideZoom (e) {
      let dom = this.$el.querySelector('.slide-dot')
      dom.addEventListener('mousedown', (e) => {
        // moveAt(e.pageX, e.pageY)
        // console.log(e)
        dom.addEventListener('mousemove', (e) => {
          moveAt(event.pageX, event.pageY)
        })
      })
      dom.addEventListener('mouseup', (e) => {
        dom.addEventListener('mousemove', null)
      })
      function moveAt (pageX, pageY) {
        console.log(pageX)
        dom.style.left = pageX + 'px'
      }
    }
  }
}
</script>

<style>
.wrapper{
  width: 500px;
  height: 500px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  background: rgba(0,0,0,0.3);
}
.wrapper canvas {
  background: #fff;
}
.controls{
  display: flex;
  width: 100%;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: center;
}
.slide {
  flex:1;
  height: 30px;
  position: relative;
}

.slide .slide-track{
  background: #F55;
  height: 10px;
  margin: 10px 0;
  border-radius: 5px;

}

.slide-dot {
  position: absolute;
  top: 5px;
  left: 5px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: rgba(22,22,22,0.5);
  transform: translate(-10px, 0);
}
.zoom-in,
.zoom-out{
  width: 80px;
  margin: 0 10px;
}
</style>
