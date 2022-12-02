<template>
  <div class="article-container">
    <!-- 导航 -->
    <van-nav-bar
      class="page-nav-bar"
      title="头条"
      left-arrow
      @click-left="onClickLeft"
    />
    <!-- 导航 -->

    <!-- 加载 -->
    <div class="main-wrap">
      <div v-if="loading" class="loading-wrap">
        <van-loading
          color="#3296fa"
          vertical
        >加载中</van-loading>
      </div>

      <div v-else-if="article.title" class="article-detail">
        <h1 class="article-title">{{article.title}}</h1>

          <van-cell
            class="user-info"
            center
            :border="false"
          >
            <van-image
              class="avatar"
              slot="icon"
              fit="cover"
              round
              :src="article.aut_photo"
            />

            <div class="user-name" slot="title">{{article.aut_name}}</div>
            <div class="publish-date" slot="label">{{article.pubdate | relativeTime}}</div>
            <follow-user
              class="follow-btn"
              v-model="article.is_followed"
              :user-id="article.aut_id"
            />
            <!-- <van-button
              v-if="article.is_followed"
              class="follow-btn"
              round
              size="small"
              :loading="followLoading"
              @click="onFollow"
            >已关注</van-button>
            <van-button
              v-else
              class="follow-btn"
              round
              type="info"
              color="#3296fa"
              size="small"
              icon="plus"
              :loading="followLoading"
              @click="onFollow"
            >关注</van-button> -->
          </van-cell>

          <div
           class="article-content markdown-body"
           v-html="article.content"
           ref="article-content"
           ></div>
          <van-divider>正文结束</van-divider>

          <!-- 评论 -->
          <comment-list
            :source="article.art_id"
            :list="commentList"
            @onload-success="totalCommentCount = $event.total_count"
            @reply-click="onReplyClick"
          />
          <!-- 评论 -->

          <div class="article-bottom">
            <van-button
              class="comment-btn"
              type="default"
              round
              size="small"
              @click="isPostShow = true"
            >写评论</van-button>
            <van-icon
              name="comment-o"
              :info="totalCommentCount"
              color="#777"
            />
            <collect-article
              class="btn-item"
              v-model="article.is_collected"
              :article-id="article.art_id"
            />
            <like-article
              class="btn-item"
              v-model="article.attitude"
              :article-id="article.art_id"
            />
            <van-icon name="share" color="#777"></van-icon>
          </div>

          <van-popup
           v-model="isPostShow"
           position="bottom"
          >
           <comment-post
             :target="article.art_id"
             @post-success="onPostSuccess"
           />
          </van-popup>

      </div>

      <!-- 加载失败 -->
      <div v-else-if="errStatus === 404" class="error-wrap">
        <van-icon name="failure" />
        <p class="text">该资源不存在或已删除！</p>
      </div>

      <div v-else class="error-wrap">
        <van-icon name="failure" />
        <p class="text">内容加载失败！</p>
        <van-button
         class="retry-btn"
         @click="loadArticle"
        >点击重试</van-button>
      </div>
      <!-- 加载失败 -->
    </div>

    <!-- 评论回复 -->
    <van-popup
      v-model="isReplyShow"
      position="bottom"
      style="height: 100%"
    >
      <comment-reply
        v-if="isReplyShow"
        :comment="currentComment"
        @close="isReplyShow = false"
      />
    </van-popup>
    <!-- 评论回复 -->
    <!-- 加载 -->
  </div>
</template>

<script>
import { getArticleById } from '@/api/article'
import { ImagePreview } from 'vant'
import FollowUser from '@/components/follow-user'
import CollectArticle from '@/components/collect-article'
import LikeArticle from '@/components/like-article'
import CommentList from './components/comment-list.vue'
import commentPost from './components/comment-post.vue'
import CommentReply from './components/comment-reply.vue'
// import { relativeTime } from 'dayjs/locale/zh-cn'

export default {
  name: 'ArticleIndex',
  components: {
    FollowUser,
    CollectArticle,
    LikeArticle,
    CommentList,
    commentPost,
    CommentReply
  },
  provide: function () {
    return {
      articleId: this.articleId
    }
  },
  props: {
    articleId: {
      type: [Number, String, Object],
      required: true
    }
  },
  data () {
    return {
      article: {},
      loading: true,
      errStatus: 0,
      followLoading: false,
      totalCommentCount: 0,
      isPostShow: false,
      commentList: [],
      isReplyShow: false,
      currentComment: {}
    }
  },
  created () {
    this.loadArticle()
  },
  methods: {
    onClickLeft () {
      this.$router.back()
    },
    async loadArticle () {
      this.loading = true
      try {
        const { data } = await getArticleById(this.articleId)
        this.article = data.data
        setTimeout(() => {
          this.previewImage()
        }, 0)
      } catch (err) {
        if (err.response && err.response.status === 404) {
          this.errStatus = 404
        }
        console.log('获取数据失败', err)
      }
      this.loading = false
    },
    previewImage () {
      const articleContent = this.$refs['article-content']
      const imgs = articleContent.querySelectorAll('img')
      const images = []
      imgs.forEach((img, index) => {
        images.push(img.src)
        img.onclick = () => {
          ImagePreview({
            images,
            startPosition: index
          })
        }
      })
    },
    onPostSuccess (data) {
      this.isPostShow = false
      this.commentList.unshift(data.new_obj)
    },
    onReplyClick (comment) {
      this.currentComment = comment
      this.isReplyShow = true
    }
  }
}
</script>

<style scoped lang="less">
@import "./github-markdown.css";
.article-container {
  .main-wrap {
    position: fixed;
    left: 0;
    right: 0;
    top: 92px;
    bottom: 88px;
    overflow-y: scroll;
    background-color: #fff;
  }
  .article-title {
    font-size: 40px;
    padding: 50px 32px;
    margin: 0;
    color: #3a3a3a;
  }
  .user-info {
     padding: 0 32px;
      .avatar {
        width: 70px;
        height: 70px;
        margin-right: 17px;
      }
      .van-cell_lable {
        margin-top: 0;
      }
      .user-name {
        font-size: 24px;
        color: #3a3a3a;
      }
      .publish-date {
        font-size: 23px;
      }

        .follow-btn {
        justify-content: center;
        width: 170px;
        height: 70px;
        // .van-button__text {
        //   display: flex;
        //   justify-content: center;
        //   align-items: center;
        //   line-height: 70px;
        //   .van-icon-plus {
        //     margin-right: 5px;
        //   }
        // }
        }
  }
  .article-content {
    padding: 55px 32px;
    /deep/ p {
      text-align: justify;
    }
  }
  .loading-wrap {
    padding: 200px 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
  }
  .error-wrap {
    padding: 200px 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
  }
  .article-bottom {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    justify-content: space-around;
    align-items: center;
    box-sizing: border-box;
    height: 88px;
    border-top: 1px solid #d8d8d8;
    background-color: #fff;
    .comment-btn {
      width: 282px;
      height: 46px;
      border: 2px solid #eee;
      font-size: 30px;
      line-height: 46px;
      color: #a7a7a7;
    }
  }
  .article-bottom {
    /deep/ .van-icon {
      font-size: 40px;
    }
    .btn-item {
      border: none;
      padding: 0;
      height: 40px;
      line-height: 40px;
      color: #777;
    }
  }
}
</style>
