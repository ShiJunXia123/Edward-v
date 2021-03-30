<template>
  <div class="hello">

    <picture-cropper
          infoText="<div>
              <div>
                最多1个文件，最大20M，支持png,jpg,jpeg格式。推荐用透明底色PNG格式。查看上传规范请点击“上传规范说明”。
              </div>
            </div>"
          :cropBoxSize="{ width: 220, height: 330 }"
          :croppedBoxSize="{ width: 80, height: 80, borderRadius: '100%' }"
          :limitSize=2
          action="http://localhost:8080/#/"
          saveBtnInfo="保存头像"
          :handleSuccess="handleSuccess"
          :proportion="9/16"
          :showPreview="false"/>
  </div>
</template>

<script>
import pictureCropper from './PictureCropper'
export default {
  components: {
    'picture-cropper': pictureCropper

  },
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods: {
    handleSuccess (response) {
      console.log(response)
      console.log(pictureCropper, 1)
    }
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
