<template>
  <div class="container">
    <TopSwiper :tops="tops"></TopSwiper>
    <Card v-for=" (book, index) in list" :key="book.id" :book="book"></Card>
    <p v-if="hasNoMoreBooks" class="nomore">没有更多书籍了~</p>
  </div>
</template>

<script>

  import Card from "@/components/Card"
  import TopSwiper from "@/components/TopSwiper"

  export default {
    components: {
      Card,
      TopSwiper
    },
    data() {
      return {
        list: [],
        tops: [],
        total: 0,
        start: 0
      }
    },
    computed: {
      hasNoMoreBooks() {
        //return this.list.length === 0 || this.start >= this.total
        return this.list.length !== 0 && this.start >= this.total
      }
    },
    methods: {
      showLoading() {
        wx.showNavigationBarLoading()
      },
      hideLoading() {
        wx.hideNavigationBarLoading()
      },
      getTop() {
        wx.cloud.callFunction({
          name: 'getTops'
        }).then(res => {
          console.log('getTops-cloud', res)
          this.tops = (res.result.tops && res.result.tops.length) ? res.result.tops : []
        })
      },
      getBooks(init) {
        if (init) {
          this.start = 0
        }
        wx.showNavigationBarLoading()
        wx.cloud.callFunction({
          name: 'getBooks',
          data: {
            start: this.start
          }
        }).then(res => {
          wx.stopPullDownRefresh()
          console.log('getBooks-cloud', res)
          if (init) {
            this.list = (res.result.books && res.result.books.length) ? res.result.books : []
          } else {
            this.list = (res.result.books && res.result.books.length) ? [...this.list, ...res.result.books] : []
          }
          this.total = res.result.total
          this.start = this.start + res.result.count
          wx.hideNavigationBarLoading()
        })
      }
    },
    mounted() {
      this.getTop()
      this.getBooks(true)
    },
    onPullDownRefresh() {
      this.getTop()
      this.getBooks(true)
    },
    onReachBottom() {
      if (this.start >= this.total) {
        return
      }
      this.getBooks()
    }
  }
</script>

<style lang="less" scoped>
  @import '~styles/variables.less';

  .container {
    padding: 0 15px;
  }

  .nomore {
    text-align: center;
    font-size: 14px;
    line-height: 1.5;
  }
</style>
