<template>
    <div class="piccrop-wrapper">
      <el-upload
        :action="action"
        :before-upload="beforeUpload"
        :show-file-list="false">
        <el-button type="primary">上传图片</el-button>
      </el-upload>
      <div class="piccrop-wrapper__div-text"><div v-html="infoText" ></div></div>
      <div class="piccrop-wrapper__wrapper">
        <div class="piccrop-wrapper__left">
          <!-- {/* 截取头像的框 */} -->
          <div
            class="piccrop-wrapper__div-img-box"
            ref="clipBoxWrapper"
            :style="{width: `${cropBoxSize.width}px`,height: `${cropBoxSize.height}px`}"
            @mouseleave="handleMouseLeave"
          >
              <crop-handler
                v-if="showPicture"
                ref="clipBox"
                :handleMouseDown="handleMouseDown"
                :clipDivSize="proportion?{...clipDivSize,height:clipDivSize.width/proportion}:clipDivSize"
                :clipWidth="clipWidth*0.5"
                :clipHeight="clipHeight*0.5"
              />
            <canvas
              ref="canvasRef"
              :width="cropBoxSize.width"
              :height="cropBoxSize.height"
            />
          </div>
          <el-button type="primary" @click="submmitCropImg">{{saveBtnInfo || "保存形象照以及头像"}}</el-button>
        </div>
        <div class="piccrop-wrapper__right">
          <div class="piccrop-wrapper__right__prelook">
            <div>
              <div
                class="piccrop-wrapper__right__uploadbox"
                :style="{
                  width: `${croppedBoxSize.width}px`,
                  height: `${croppedBoxSize.height}px`,
                  borderRadius: `${croppedBoxSize.borderRadius}`,
                }"
              >
                <img
                  :style="{ position: 'relative' }"
                  ref="preview"
                  :src="previewUrl"
                  :width="(croppedBoxSize.width / clipDivSize.width) * cropBoxSize.width"
                  :height="(croppedBoxSize.height / clipDivSize.height) * cropBoxSize.height"
                  class="piccrop-wrapper__img-yulan"
                />
                <div
                  id="upload-img-box"
                  class="piccrop-wrapper__right__img-box"
                />
                <div id="img-box" class="piccrop-wrapper__right__img-box" >
                  <img id="newImg" :width="cropBoxSize.width" :height="cropBoxSize.height" ref='newImg' :src="previewUrl"/>
                </div>
              </div>
              <p class="piccrop-wrapper__text-color">查看预览图</p>
            </div>
            <div id="crop-img" class="piccrop-wrapper__right__cropimg" />
          </div>
        </div>
      </div>
    </div>
</template>
<script>
import { getAttr, judgeBundle, setAttr, setNewSize } from './util'
import { ONEZORE } from './const'
import defaultAvatar from './image/default-avatar.png'
import CropHandler from './CropHandler'
export default {
  components: { 'crop-handler': CropHandler },
  props: {
    action: String, // 上床url
    infoText: String, // 注意事项
    cropBoxSize: {type: Object, // 左侧截图框大小
      default: () => ({ width: 164,
        height: 210})},
    croppedBoxSize: {type: Object, // 被截图的框大小
      default: () => ({ width: 54,
        height: 54,
        borderRadius: '100%'})},
    clipDivSize: {type: Object, // 截图矩形初始大小
      default: () => ({ width: 100,
        height: 100})},
    limitSize: Number,
    fixedCropWidth: Number, // 无论截图框大小，自定义最终转化成图片的像素宽
    fixedCropHeight: Number, // 无论截图框大小，自定义最终转化成图片的像素宽
    saveBtnInfo: String,
    handleSuccess: Function,
    proportion: Number, // 截图框比例，默认是1
    clipDivWidth: Number,
    clipDivHeight: Number
  },
  data () {
    return {
      canvasRefContext: null,
      previewRefContext: null,
      clipBoxWrapperRef: null,
      scale: { scaleX: 1, scaleY: 1 },
      clipBoxRef: null,
      previewRef: null,
      previewUrl: '',
      ONEZORE: ONEZORE,
      successData: '', // 上传成功后返回信息,
      showPicture: false, // 是否成功展示图片
      //
      //
      isShowUp: false,
      clipWidth: 0,
      clipHeight: 0,
      cropBoxSize: {
        width: 400,
        height: 300
      },
      clipDivSize: {
        width: 0,
        height: 0
      },
      croppedBoxSize: {
        width: 400,
        height: 300
      }
    }
  },
  methods: {
    initCanvas () {
      this.clipBoxWrapperRef = this.$refs.clipBoxWrapper
      this.previewRef = this.$refs.preview
      this.canvas = this.$refs.canvasRef
      this.canvasRefContext = this.canvas.getContext('2d')
    },
    // 上传图片时，创建img，画到canvas中
    showImgInCanvas  (url) {
      this.previewUrl = url
      const img = this.$refs.newImg
      // 之前用的是上传之后接口返回的超链，处理完画布污染有跨域问题

      // 图片加载完成后展示到canvas
      img.onload = () => {
        this.canvasRefContext.drawImage(img, 0, 0, this.cropBoxSize.width, this.cropBoxSize.height)

        this.scale = {
          scaleX: this.cropBoxSize.width / img.naturalWidth,
          scaleY: this.cropBoxSize.height / img.naturalHeight
        }
      }
    },

    // 上传前处理避免上传成功后再引用图片跨域
    beforeUpload (file) {
      const that = this
      if (this.limitSize) {
        if (file.size / 1024 / 1024 > that.limitSize) {
          that.$message.error(`图片大小不能超过${that.limitSize}M`)
          return false
        }
        this.showPicture = true
        const reader = new FileReader()
        reader.readAsDataURL(file)

        reader.onload = (event) => {
          that.showImgInCanvas(event.target.result)
          this.clipBoxRef = this.$refs.clipBox.$el
        }

        return false
      }

      return false
    },

    /**
     * @description  判断合理后最终赋值
     * @param newCropSize:NewCropSize;
     * */
    setCropSize  (newCropSize, clipBoxArea) {
      // 获取当前截图框的宽高
      const cropSize = {
        width: parseInt(getAttr(clipBoxArea, 'width')),
        height: parseInt(getAttr(clipBoxArea, 'height'))
      }

      const previewNew = this.previewRef

      // 赋值
      if (newCropSize.cropWidth) {
        setAttr(clipBoxArea, 'width', `${newCropSize.cropWidth}px`)
        setAttr(
          previewNew,
          'width',
          `${
            (this.croppedBoxSize.width / newCropSize.cropWidth) * this.cropBoxSize.width
          }px`
        )
      }

      if (newCropSize.cropHeight) {
        setAttr(clipBoxArea, 'height', `${newCropSize.cropHeight}px`)

        setAttr(
          previewNew,
          'height',
          `${
            (this.croppedBoxSize.height / newCropSize.cropHeight) * this.cropBoxSize.height
          }px`
        )
      }

      if (newCropSize.cropLeft || newCropSize.cropLeft === 0) {
        setAttr(clipBoxArea, 'left', `${newCropSize.cropLeft}px`)

        setAttr(
          previewNew,
          'left',
          `-${(this.croppedBoxSize.width / cropSize.width) * newCropSize.cropLeft}px`
        )
      }

      if (newCropSize.cropTop || newCropSize.cropTop === 0) {
        setAttr(clipBoxArea, 'top', `${newCropSize.cropTop}px`)

        setAttr(
          previewNew,
          'top',
          `-${(this.croppedBoxSize.height / cropSize.height) * newCropSize.cropTop}px`
        )
      }
    },

    /**
     * @description 鼠标在截取图片div上按下的事件
     * @param event:MouseEvent
     * */
    handleMouseDown  (event) {
      const oEvent = event || window.event

      // 当前裁剪区域位置信息
      const clipBoxAttr = {
        left: getAttr(this.clipBoxRef, 'left'),
        top: getAttr(this.clipBoxRef, 'top'),
        width: getAttr(this.clipBoxRef, 'width'),
        height: getAttr(this.clipBoxRef, 'height')
      }

      // 判断鼠标位置
      let mouse = {
        mouseX: oEvent.clientX,
        mouseY: oEvent.clientY,
        mouseToX: oEvent.clientX,
        mouseToY: oEvent.clientY
      }

      //! 一旦鼠标按下，那么cursor的类型就不变了
      const cursor = getAttr(oEvent.target, 'cursor')
      const that = this
      document.onmousemove = ev => {
      // document.onmousemove = ev => {
        const toEvent = ev || window.event

        mouse = {
          ...mouse,
          mouseToX: toEvent.clientX - mouse.mouseX,
          mouseToY: toEvent.clientY - mouse.mouseY
        }

        const newCropSize = setNewSize(toEvent, cursor, clipBoxAttr, mouse, this.proportion)
        const newCropSizeCopy = judgeBundle(
          newCropSize,
          that.clipBoxRef,
          that.clipBoxWrapperRef,
          this.proportion
        )

        this.setCropSize(newCropSizeCopy, that.clipBoxRef)
      }

      document.onmouseup = () => {
        // 移动事件取消
        document.onmousemove = null

        // 阻止默认事件
        oEvent.preventDefault()
      }

      oEvent.preventDefault()
    },
    handleMouseLeave (event) {
      // const oEvent = event || window.event
      document.removeEventListener('mousemove', this.clipBoxWrapperRef)
    },

    // 截取图片后，上传
    croppedImgUpload (data) {
      const that = this
      const arr = data.split(',')
      // 拿到类型
      const mime = arr[0].match(/:(.*?);/)[1]
      // 拿到图片类型
      const bstr = atob(arr[1])
      let { length } = bstr
      const u8arr = new Uint8Array(length)

      while (length--) {
        u8arr[length] = bstr.charCodeAt(length)
      }

      // 转化文件格式,此处文件名称必须添加文件类型后缀
      const file = new File([u8arr], `photo.${mime.split('/')[1]}`, {
        type: mime
      })

      const formData = new FormData()
      formData.append('file', file)

      fetch(that.action, {
        body: formData,
        method: 'POST'
      })
        .catch((error) => that.$message.error(error))
        .then((response) => response.json())
        .then((response) => {
          that.handleSuccess(response)

          return that.$message({
            message: '图片上传成功！',
            type: 'success'
          })
        })
    },

    // 获取截取图片
    cropImg  (image) {
      // canvas转化成图片
      document.querySelector('#upload-img-box').innerHTML = ''
      const newCanvas = document.createElement('canvas')
      newCanvas.width = this.fixedCropWidth || this.croppedBoxSize.width
      newCanvas.height = this.fixedCropHeight || this.croppedBoxSize.height
      document.querySelector('#upload-img-box').appendChild(newCanvas)

      const newCtx = newCanvas.getContext('2d')

      // canvas转化为图片
      const newImage = new Image()
      newImage.setAttribute('crossOrigin', 'anonymous')

      newImage.onload = async () => {
        // 根据缩放比例确定截取位置
        newCtx.drawImage(
          image,
          parseFloat(this.clipBoxRef.style.left || this.clipWidth / 10) / this.scale.scaleX || 0,
          parseFloat(this.clipBoxRef.style.top || this.clipHeight / 10) / this.scale.scaleY || 0,
          parseInt(this.clipBoxRef.style.width) / this.scale.scaleX,
          parseInt(this.clipBoxRef.style.height) / this.scale.scaleY,
          0,
          0,
          this.fixedCropWidth || this.croppedBoxSize.width,
          this.fixedCropWidth || this.croppedBoxSize.height
        )

        const data = newCanvas.toDataURL()
        this.croppedImgUpload(data)
      }

      // 使用ref拿不到toDataURL方法，暂时用创建元素的方法来实现
      newImage.src = defaultAvatar
      document.querySelector('#crop-img').appendChild(newImage)
    },

    // 确认截取图片
    submmitCropImg () {
      this.cropImg(document.querySelector('#newImg'))
    }
  },
  mounted () {
    this.initCanvas()
    // 鼠标抬起
    document.addEventListener('mouseup', ev => {
      const oEvent = ev || window.event

      // 移动事件取消
      this.clipBoxRef && (this.clipBoxRef.onmousemove = null)

      // 阻止默认事件
      oEvent.preventDefault()
    })
  },
  beforeDestroy () {
    window.removeEventListener('mouseup', this.handleResize, false)
  }
}
</script>
<style lang='less'>
.piccrop-wrapper {
  padding: 20px;

  .displayNone {
    display: none;
  }

  &__wrapper {
    display: flex;
    width: 100%;
    justify-content: space-around;
    border-radius: 4px;
    padding: 20px;
  }

  &__div-text {
    color: #757c82;
    line-height: 30px;
  }

  &__left {
    text-align: center;
  }

  &__img-empty {
    position: absolute;
    top: 40%;
    left: 40%;
  }

  &__div-img-box {
    position: relative;
    background-color: #cae0f3;
    width: auto;
    height: auto;
    border-radius: 4px;
    margin: 5px 0;
  }

  &-handler {
    position: absolute;
    top: 0;
    border: 1px dashed #eee;
    cursor: move;
    left: 10%;
    top: 10%;

    &-bg {
      height: 100%;
      width: 100%;
      position: absolute;
      background: #000;
      opacity: 0.03;
    }

    &-squire {
      width: 7px;
      height: 7px;
      border: 1px solid #26d3bc;
      position: absolute;
      background-color: #26d3bc;
      opacity: 0.5;
    }
  }

  &__right {
    padding: 20px;

    &__prelook {
      display: flex;
      margin-top: 60px;
      justify-content: space-around;
      p {
        text-align: center;
      }
    }

    &__img-box {
      visibility: hidden;
    }

    &__uploadbox {
      width: auto;
      height: auto;
      text-align: center;
      background-color: #dff8f4;
      border-radius: 4px;
      overflow: hidden;
    }

    &__cropimg {
      position: absolute;
      visibility: hidden;
    }

    .picCropWrapper__img-empty {
      position: relative;
      top: 20%;
      left: 0%;
    }
  }

  &__text-color {
    color: #26d3bc;
  }
}

</style>
