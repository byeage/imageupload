<template>
  <div class="image-upload">
    <div class="change">
      <span>更换头像</span>
      <input type="file"  id="upload-avatar" hidden name="avatar" accept="image/*" @change="fileUpload($event)"/>
      <label for="upload-avatar">上传本地照片</label>
    </div>
    <div class="wrapper">
      <canvas width="300" height="300"></canvas>
    </div>
    <div class="controls">
      <div class="zoom-out">
        <button type="button" @click.prevent="zoomOut">Out</button>
      </div>
      <div class="scale">
        <input type="range" v-model="scale" step="1" min="0" :max="maxScale">
      </div>
      <div class="zoom-in">
        <button type="button" @click.prevent="zoomIn">In</button>
      </div>
    </div>
    <div class="sure">
      <button class="sure" @click.prevent="exportData">sure</button>
    </div>
  </div>
</template>
<script>
export default {
  name: 'image-upload',
  props: ['image', 'imageData', 'fileUpload'],
  data () {
    return {
      scale: 0,
      maxScale: 100,
      percent: 1 / (3 - 0.1) * 100,
      imageHandler: undefined
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
      this.imageHandler.doZoom(this.scale)
    },
    imageData () {
      this.render()
    }
  },
  mounted () {
    this.render()
  },
  methods: {
    render () {
      let canvas =  this.$el.querySelector('canvas')

      let image = new Image()
      image.src = this.imageData
      image.onload = () => {
        let imageClipUpload = new ImageClipUpload(canvas, image)
        this.imageHandler = imageClipUpload
        imageClipUpload.doDraw()
      }


      function ImageClipUpload(canvas, image) {
        this.rotate = Math.PI / 180 * 90
        this.maxScale = 10
        this.minScale = 0.1
        this.image = image
        this.transform = {
          x: 0,
          y: 0,
          scale: 1,
          rotate: 0
        }
        this.move = {
          x: 0,
          y: 0
        }

        this.ctx = canvas.getContext('2d')
        this.imageWidth = image.width
        this.imageHeight = image.height
        this.containerWidth = canvas.clientWidth
        this.containerHeight = canvas.clientHeight
        this.clipRadius = Math.min(canvas.clientHeight,canvas.clientHeight) / 2
        let self = this
        let sx = 0
        let sy= 0
        let ox = 0
        let oy = 0

        canvas.onmousedown =  (e) => {
          console.log('mousedown')
          sx = e.pageX
          sy = e.pageY
          ox = self.move.x
          oy = self.move.y
          document.addEventListener('mousemove', mousemove)
          document.addEventListener('mouseup', mouseup)
        }

        function mousemove (e) {
          let x = e.pageX
          let y = e.pageY
          let mx = x - sx
          let my = y - sy
          self.move.x = ox + mx
          self.move.y = oy + my
          self.doDraw()

        }
        function mouseup () {
          console.log('mouseup')
          document.removeEventListener('mousemove', mousemove)
          document.removeEventListener('mouseup', mouseup)
        }

      }

      ImageClipUpload.prototype.doDraw = function () {
        this.ctx.save()
        this.ctx.clearRect(0, 0, this.containerWidth, this.containerHeight)
        this.ctx.restore()
        this.ctx.save()
        this.ctx.setTransform(1,0,0,1,0,0)
        this.ctx.scale(this.transform.scale, this.transform.scale)
        this.ctx.translate(this.transform.x + this.move.x, this.transform.y + this.move.y)
        this.ctx.drawImage(this.image,
          0,
          0,
          this.containerWidth,
          this.containerHeight)
        this.ctx.restore()
        this.ctx.save()
        this.ctx.fillStyle = 'rgba(0,0,0,0.5)'
        this.ctx.beginPath()
        this.ctx.arc(this.containerWidth/2, this.containerHeight/2, this.clipRadius / 1.01, 0, Math.PI * 2)
        this.ctx.rect(this.containerWidth,0, -this.containerWidth, this.containerHeight)
        this.ctx.fill()
        this.ctx.restore()
      }

      ImageClipUpload.prototype.doZoomOut = function () {}
      ImageClipUpload.prototype.doZoomIn = function () {}
      ImageClipUpload.prototype.doZoom = function (scale) {

        this.transform.scale = scale / 100 + 1

        let x = this.containerWidth / 2
        let y = this.containerHeight / 2

        let w = this.containerWidth * this.transform.scale
        let h = this.containerHeight * this.transform.scale

        this.transform.x = x - w / 2
        this.transform.y = y - h / 2
        this.doDraw()
      }
      ImageClipUpload.prototype.doTransform = function () {}
      ImageClipUpload.prototype.doExportData = function () {
        let canvas = document.createElement('canvas')
        canvas.style.width = this.containerWidth
        canvas.style.height = this.containerHeight
        let context = canvas.getContext('2d')
        context.setTransform(1,0,0,1,0,0)
        context.scale(this.transform.scale, this.transform.scale)
        context.translate(this.transform.x + this.move.x, this.transform.y + this.move.y)
        context.drawImage(this.image,
          0,
          0,
          this.containerWidth,
          this.containerHeight)
        let base64 = canvas.toDataURL()
        let formData =  new FormData()
        formData.append('file', base64)
        console.log(formData.get('file'))
      }
    },
    zoomIn () {
      let scale = this.scale
      scale ++
      if (scale > this.maxScale) {
        scale = this.maxScale
      }
      this.scale = scale
    },
    zoomOut () {
      let scale = this.scale
      scale --
      if (scale < 0) {
        scale = 0
      }
      this.scale = scale
    },
    exportData () {
      this.imageHandler.doExportData()
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
  margin: 0 auto;
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
.scale {
  flex: 1;
  margin:  0 15px;
}

.scale input {
  width: 100%;
}

.zoom-in,
.zoom-out{
  width: 80px;
  margin: 0 10px;
}

input[type=range] {
  -webkit-appearance: none;
  margin: 18px 0;
  width: 100%;
}
input[type=range]:focus {
  outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
  width: 100%;
  height: 5px;
  cursor: pointer;
  animate: 0.2s;
  background: #848486;
  border-radius: 2.5px;
}
input[type=range]::-webkit-slider-thumb {
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #46B8D9;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -5px;
}
input[type=range]:focus::-webkit-slider-runnable-track {

}
/* moz */
input[type=range]::-moz-range-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  animate: 0.2s;
  box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
  background: #3071a9;
  border-radius: 1.3px;
  border: 0.2px solid #010101;
}
input[type=range]::-moz-range-thumb {
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #46B8D9;
  cursor: pointer;
  margin-top: -5px;
}
/* ms */
input[type=range]::-ms-track {
  width: 100%;
  height: 8.4px;
  cursor: pointer;
  animate: 0.2s;
  background: transparent;
  border-color: transparent;
  border-width: 16px 0;
  color: transparent;
}
input[type=range]::-ms-fill-lower {
  background: #2a6495;
  border: 0.2px solid #010101;
  border-radius: 2.6px;
  box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
}
input[type=range]::-ms-fill-upper {
  background: #3071a9;
  border: 0.2px solid #010101;
  border-radius: 2.6px;
  box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
}
input[type=range]::-ms-thumb {
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #46B8D9;
  cursor: pointer;
  margin-top: -5px;
}
input[type=range]:focus::-ms-fill-lower {
  background: #3071a9;
}
input[type=range]:focus::-ms-fill-upper {
  background: #367ebd;
}
</style>
