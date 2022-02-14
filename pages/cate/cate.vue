<template>
  <view>
    <!-- 使用自定义的搜索组件 -->
    <my-search @click="gotoSearch"></my-search>
    
    <view class="scroll-view-container">
      <!-- 左侧的滑动区域 -->
      <scroll-view class="left-scroll-view" scroll-y="true" :style="{height: wh + 'px'}">
        <block v-for="(item, i) in cateList" :key="i">
          <view :class="['left-scroll-view-item', i === active ? 'active' : '']" @click="activeChanged(i)">{{item.cat_name}}</view>
        </block>
      </scroll-view>
      
      <!-- 右侧的滑动区域 -->
      <scroll-view scroll-y="true" :style="{height: wh + 'px'}" :scroll-top="scrollTop">
        <view class="cate-lv2" v-for="(item2, i2) in cateLevel2" :key="i2">
          <!-- 二级分类商品标题 -->
          <view class="cate-lv2-title">/ {{item2.cat_name}} /</view>
          
          <!-- 二级分类下的三级分类列表 -->
          <view class="cate-lv3-list">
            <!-- 三级分类的 item 项 -->
            <view class="cate-lv3-item" v-for="(item3, i3) in item2.children" :key="i3" @click="gotoGoodsList(item3)">
              <!-- 三级分类的图片 -->
              <image :src="item3.cat_icon"></image>
              <!-- 三级分类的文本 -->
              <text>{{item3.cat_name}}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 当前设备的可用高度
        wh: 0,
        cateList: [],
        active: 0,
        
        // 二级分类的列表
        cateLevel2: [],
        
        // 滚动条距离顶部的位置
        scrollTop: 0
      };
    },
    onLoad() {
      // 返回设备的可用高度
      const sysInfo = uni.getSystemInfoSync()
      this.wh = sysInfo.windowHeight - 50
      
      // 获取分类列表的数据
      this.getCateList()
    },
    methods: {
      // 获取分类列表的数据
      async getCateList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/categories')
        // 请求失败
        if (res.meta.status !== 200) return uni.$showMsg()
        // 请求成功并为一级分类和二级分类赋值
        this.cateList = res.message
        this.cateLevel2 = res.message[0].children
      },
      
      // 更改激活项并重新为二级分类赋值，并重置滚动条的位置
      activeChanged(i) {
        this.active = i
        this.cateLevel2 = this.cateList[i].children
        this.scrollTop = this.scrollTop === 0 ? 1 : 0
      },
      
      // 跳转到商品列表页面
      gotoGoodsList(item3) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?cid=' + item3.cat_id
        })
      },
      
      // 点击搜索框事件
      gotoSearch() {
        uni.navigateTo({
          url: '/subpkg/search/search'
        })
      }
    }
  }
</script>

<style lang="scss">
  .scroll-view-container {
    display: flex;
    
    .left-scroll-view {
      width: 120px;
      
      .left-scroll-view-item {
        background-color: #F7F7F7;
        line-height: 60px;
        text-align: center;
        font-size: 12px;
        
        // 表示元素既包含 left-scroll-view-item 和 active 类名
        &.active {
          background-color: #FFFFFF;
          position: relative;
          
          // 伪元素，在该区域的前面(before)加上元素
          &::before {
            content: ' ';
            display: block;
            width: 3px;
            height: 30px;
            background-color: #C00000;
            position: absolute;
            // transform 的设置要配合该元素和父元素的 position 使用，效果会更好
            // translateY(50%) 表示Y轴平移自身高度的一半
            transform: translateY(50%);
          }
        }
      }
    }
  }
  
  .cate-lv2-title {
    font-size: 12px;
    font-weight: bold;
    text-align: center;
    padding: 15px 0;
  }
  
  .cate-lv3-list {
    display: flex;
    flex-wrap: wrap;
    
    .cate-lv3-item {
      width: 33.33%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      margin-bottom: 10px;
      
      image {
        width: 60px;
        height: 60px;
      }
      
      text {
        font-size: 12px;
      }
    }
  }
</style>
