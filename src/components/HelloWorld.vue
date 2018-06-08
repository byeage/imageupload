<template>
  <div class="image-upload">
    <input type="file" name="image" accept="image/*" @change="fileUpload($event)"/>
    <button>Upload</button>
    <div>
      <image-upload v-if="showImageUpload" :image="image" :image-data="imageData"></image-upload>
    </div>
  </div>
</template>

<script>
import ImageUpload from './ImageUpload.vue'
export default {
  name: 'HelloWorld',
  data () {
    return {
      image: {},
      imageData: {},
      showImageUpload: false
    }
  },
  methods: {
    fileUpload (e) {
      this.image = e.target.files[0]
      let reader = new FileReader()
      reader.onload = () => {
        this.imageData = reader.result
        this.showImageUpload = true
      }
      reader.readAsDataURL(e.target.files[0])
    }
  },
  components: {
    ImageUpload
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
