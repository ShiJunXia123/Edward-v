<template>
     <div
      id="clip-div"
      class='piccrop-wrapper-handler'
      @mousedown="handleMouseDown($event)"
      :style="{
        width: `${clipDivSize.width}px`,
        height: `${clipDivSize.height}px`
      }"
    >
      <div class="piccrop-wrapper-handler-bg" />
      <div :class="this.squire" :style="{ left: 0, cursor: 'nw-resize' }" />
      <div :class="this.squire" :style="{ left: '50%', cursor: 'n-resize' }" />
      <div :class="this.squire" :style="{ right: 0, cursor: 'ne-resize' }" />
      <div :class="this.squire" :style="{ top: '50%', cursor: 'w-resize' }" />
      <div :class="this.squire" :style="{ right: 0, top: '50%', cursor: 'e-resize' }"/>
      <div :class="this.squire" :style="{ left: 0, bottom: 0, cursor: 'sw-resize' }" />
      <div :class="this.squire" :style="{ left: '50%', bottom: 0, cursor: 's-resize' }" />
      <div :class="this.squire" :style="{ right: 0, bottom: 0, cursor: 'se-resize' }" />
    </div>
</template>
<script>

export default {
  name: 'cropHandler',
  props: {
    handleMouseDown: Function, clipDivSize: Object
  },
  data () {
    return {squire: 'piccrop-wrapper-handler-squire'}
  },

  mounted () {
  // 多处使用className，常量定义

  }

}
</script>
