<template>
  <div>
    <!-- 输入框 -->
    <div class="search">
      <div class="search-content">
        <!-- 搜索输入框 -->
        <div class="search-input">
          <icon type="search" size="16"/>
          <input
            @confirm="confirmHandle"
            @input="inputHandle"
            v-model="keyword"
            placeholder="请输入关键字"
          >
        </div>
        <button @click="cancelHandle" v-if="keyword" class="cancel">取消</button>
        <!-- 搜索结果 -->
        <div v-if="keyword" class="search-modal">
          <div
            :key="item.goods_id"
            v-for="item in searchResult"
            class="search-item"
          >{{item.goods_name}}</div>
        </div>
      </div>
    </div>
    <!-- 搜索历史关键字 -->
    <div class="history">
      <h4>搜索历史</h4>
      <icon @click="clearHistory" type="clear" size="16"/>
    </div>
    <div class="history-list">
      <div :key="index" v-for="(item,index) in keywordHistory" class="history-item">
        <navigator :url="'/pages/search_list/main?query=' + item">
          {{item}}
        </navigator>
      </div>
    </div>
  </div>
</template>

<script>
import request from '../../utils/request.js'
export default {
  data () {
    return {
      keyword: '',
      searchResult: [],
      isLoading: false,
      timer: null,
      keywordHistory: mpvue.getStorageSync('keyword') || []
    }
  },
  methods: {
    getUrl (kw) {
      // console.log(kw);
      return '/pages/search_list/main?query=' + kw
    },
    clearHistory () {
      // 清空搜索关键字的历史信息
      // 清空的是本地存储的数据，并不会影响data中的数据
      mpvue.clearStorageSync()
      // 清空data中的数据
      this.keywordHistory = []
    },
    confirmHandle () {
      // 当回车的时候，记录关键字到本地存储
      this.keywordHistory.unshift(this.keyword)
      // 进行数组去重
      let kwh = [...new Set(this.keywordHistory)]
      // 把最新的数据覆盖到本地存储中
      mpvue.setStorageSync('keyword', kwh)
      // console.log(mpvue.getStorageSync('keyword'))
      // 重新更新页面数据
      this.keywordHistory = kwh
      // 跳转到商品列表页面
      mpvue.navigateTo({
        url: '/pages/search_list/main?query=' + this.keyword
      })
    },
    cancelHandle () {
      // 清空输入框内容
      this.keyword = ''
    },
    async inputHandle () {
      // 根据输入的关键字，调用后台接口查询匹配的数据
      // 控制请求的频率(节流)：输入多个字符，但是只发送一次请求
      // 控制是否发送请求
      if (this.isLoading) {
        // 终止后续代码的执行，即终止请求
        return
      }
      // isLoading变成true之后，就阻止后续请求
      this.isLoading = true
      this.timer = setTimeout(async () => {
        // 关闭之前的定时任务
        clearTimeout(this.timer)
        let res = await request('goods/qsearch', 'get', {
          query: this.keyword
        })
        this.searchResult = res.data.message
        // 重新打开发送请求的开关
        this.isLoading = false
      }, 1000)
    }
  }
}
</script>

<style scoped lang='scss'>
@import 'main.scss';
</style>
