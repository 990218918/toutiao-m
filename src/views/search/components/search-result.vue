<template>
<div class="search-result">
  <van-list
    v-model="loading"
    :finished="finished"
    finished-text="没有更多了"
    :error.sync="error"
    error-text="加载失败，请稍后重试"
    @load="onLoad"
  >
    <van-cell
     v-for="(article, index) in list"
     :key="index"
     :title="article.title"
     />
  </van-list>
</div>
</template>

<script>
import { getSearchResult } from '@/api/search'
// import { connected } from 'process'

export default {
  name: 'SearchResult',
  props: {
    searchText: {
      type: String,
      required: true,
      page: 1,
      perPage: 20
    }
  },
  data () {
    return {
      list: [],
      loading: false,
      finished: false,
      error: false
    }
  },
  methods: {
    async onLoad () {
      try {
        const { data } = await getSearchResult({
          page: this.page,
          per_page: this.perPage,
          q: this.searchText
        })
        const { results } = data.data
        this.list.push(...results)
        this.loading = false
        if (results.length) {
          this.page++
        } else {
          this.finished = true
        }
      } catch (err) {
        this.error = true
        this.loading = false
      }
    }
  }
}
</script>

<style scoped lang="less">

</style>
