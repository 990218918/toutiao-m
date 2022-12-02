<template>
  <div class="article-list">
    <van-pull-refresh
     v-model="isreFreshLoading"
     :success-text="refreshSuccessText"
     success-duration="1500"
     @refresh="onRefresh"
    >
        <van-list
            v-model="loading"
            :finished="finished"
            finished-text="没有更多了"
            :error.sync="error"
            error-text="请求失败，点击重新加载"
            @load="onLoad"
            >
            <article-item
             v-for="(article, index) in list"
             :key="index"
             :article="article"
            />
            <!-- <van-cell
            v-for="(article, index) in list"
            :key="index"
            :title="article.title"
            /> -->
        </van-list>
    </van-pull-refresh>
  </div>
</template>

<script>
import { getArticles } from '@/api/article'
import ArticleItem from '@/components/article-item'

export default {
  name: 'ArticleList',
  components: {
    ArticleItem
  },
  props: {
    channel: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      list: [],
      loading: false,
      finished: false,
      timestamp: null,
      error: false,
      isreFreshLoading: false,
      refreshSuccessText: '刷新成功'
    }
  },
  methods: {
    async onLoad () {
      try {
        const { data } = await getArticles({
          channel_id: this.channel.id,
          timestamp: this.timestamp || Date.now()
        })
        const { results } = data.data
        this.list.push(...results)
        this.loading = false
        if (results.length) {
          this.timestamp = data.data.pre_timestamp
        } else {
          this.finished = true
        }
      } catch (err) {
        console.log('请求失败', err)
      }
    },
    async onRefresh () {
      try {
        const { data } = await getArticles({
          channel_id: this.channel.id,
          timestamp: Date.now()
        })
        const { results } = data.data
        this.list.unshift(...results)
        this.isreFreshLoading = false
        this.refreshSuccessText = `刷新成功，更新了${results.length}条数据`
      } catch (err) {
        this.isreFreshLoading = false
        this.refreshSuccessText = '刷新失败'
      }
    }
  }
}
</script>

<style scoped lang="less">
.article-list {
    height: 79vh;
    overflow-y: auto;
}
</style>
