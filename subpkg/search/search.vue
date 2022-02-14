<template>
  <view>
    <view class="search-box">
      <uni-search-bar placeholder="请输入搜索内容" @input="input" :radius="100" cancelButton="none"></uni-search-bar>
    </view>
    
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clean"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer: null,
        kw: '',
        // 搜索的结果列表
        searchResults: [],
        // 搜索历史
        historyList: []
      };
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      // 搜索框输入事件的处理函数
      input(e) {
        // 用户输入时，复位计时器
        clearTimeout(this.timer)
        // 当用户 0.5 秒内没有输入时，打印输入的字符串
        this.timer = setTimeout(() => {
          this.kw = e.valueOf()
          this.getSearchList()
        }, 500)
      },
      
      // 获取搜索结果
      async getSearchList() {
        // 判断搜索关键词是否为空
        if (this.kw.length === 0) {
          this.searchResults = []
          return
        }
        
        const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: this.kw })
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
        
        // 保存搜索记录
        this.saveSearchHistory()
      },
      
      // 跳转到商品详情页
      gotoDetail(item) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      },
      
      // 保存搜索记录
      saveSearchHistory() {
        // 添加搜索记录到数组，set 可以消重
        const set = new Set(this.historyList)
        set.delete(this.kw)
        set.add(this.kw)
        this.historyList = Array.from(set)
        
        // 对搜索历史数据进行持久化的存储
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      
      // 清空搜索记录
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', '[]')
      },
      
      // 跳转到商品列表页面
      gotoGoodsList(kw) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    },
    
    computed: {
      histories() {
        return [...this.historyList].reverse()
      }
    }
  }
</script>

<style lang="scss">

  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }
  
  .sugg-list {
    padding: 0px 5px;
    
    .sugg-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      
      .goods-name {
        // 只允许单行文本
        white-space: nowrap;
        // 隐藏多余部分
        overflow: hidden;
        // 超出的部分用 ... 替代
        text-overflow: ellipsis;
      }
    }
  }
  
  .history-box {
    padding: 0px 5px;
    
    .history-title {
      display: flex;
      justify-content: space-between;
      height: 40px;
      align-items: center;
      font-size: 13px;
      border-bottom: 1px solid #efefef ;
    }
    
    .history-list {
      margin-top: 5px;
      display: flex;
      flex-wrap: wrap;
      
      .uni-tag {
        margin-right: 5px;
      }
    }
  }

</style>
