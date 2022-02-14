<template>
  <view class="my-search-container" :style="{ 'background-color': bgcolor }" @click="searchBoxHandler">
    <view class="my-search-box" :style="{ 'border-radius': radius + 'px' }">
      <!-- 使用 uni-ui 提供的图标组件 -->
      <uni-icons type="search" size="17"></uni-icons>
      <text class="placeholder">搜索</text>
    </view>
  </view>
</template>

<script>
  export default {
    name:"my-search",
    props: {
      // 背景颜色
      bgcolor: {
        type: String,
        default: '#C00000'
      },
      // 圆角尺寸
      radius: {
        type: Number,
        default: 18 // px
      }
    },
    data() {
      return {
        
      };
    },
    methods: {
      searchBoxHandler() {
        this.$emit('click')
      }
    }
  }
</script>

<style lang="scss">

.my-search-container {
  // background-color: #C00000;
  height: 50px;
  display: flex;
  align-items: center;
  padding: 0px 10px;
  
  .my-search-box {
    // border-radius: 18px;
    height: 36px;
    background-color: #FFFFFF;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    
    .placeholder {
      font-size: 15px;
      margin-left: 5px;
    }
  }
}

</style>
