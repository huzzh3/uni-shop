<template>
  <view>
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i" @click="gotoDetail(goods)">
        <my-goods :goods="goods"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 请求参数对象
        queryObj: {
          query: '',
          cid: '',
          pagenum: 1,
          pagesize: 10
        },
        // 请求的商品列表
        goodsList: [],
        total: 0,
        // 节流阀
        isloading: false
      };
    },
    onLoad(options) {
      // 通常要考虑对象为空的赋值的问题
      this.queryObj.query = options.query || ''
      this.queryObj.cid = options.cid || ''

      // 对请求的对象赋值完了之后，我们通过函数来请求对象
      this.getGoodsList()
    },
    methods: {
      // 获取商品列表数据的方法
      async getGoodsList(cb) {
        // 打开节流阀
        this.isloading = true
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)

        // 此时请求已经结束关闭节流阀
        this.isloading = false
        
        // 根据是否有回调函数来进行回调函数的执行
        cb && cb()

        // 若请求失败，则弹窗告诉用户请求失败了
        if (res.meta.status !== 200) return uni.$showMsg()

        // 请求成功，则对数据进行赋值
        this.goodsList = [...this.goodsList, ...res.message.goods]
        this.total = res.message.total
      },
      gotoDetail(goods) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods.goods_id
        })
      }
    },
    onReachBottom() {
      if ((this.queryObj.pagenum - 1) * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')
      
      // 根据节流阀来控制是否获取数据
      if (this.isloading) return
      
      // 让页面值自增
      this.queryObj.pagenum++
      this.getGoodsList()
    },
    onPullDownRefresh() {
      // 重置关键数据
      this.queryObj.pagenum = 1
      this.total            = 0
      this.isloading        = false
      this.goodsList        = []
      
      // 重新发起请求
      this.getGoodsList(() => uni.stopPullDownRefresh())
    },
  }
</script>

<style lang="scss">

</style>
