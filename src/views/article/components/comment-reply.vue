<template>
  <div class="comment-reply">
    <van-nav-bar
        :title="comment.reply_count > 0 ? `${comment.reply_count}条回复` : '暂无回复'"
    >
      <van-icon
       slot="left"
       name="cross"
       @click="$emit('close')"
       />
    </van-nav-bar>
    <!-- 当前评论 -->
    <div class="scroll-wrap">
        <comment-item
            :comment="comment"
        />
            <!-- 当前评论 -->

            <!-- 评论回复 -->
        <van-cell title="全部回复" />
        <comment-list
          :list="commentList"
          :source="comment.com_id"
          type="c"
        />
    </div>

    <div class="post-wrap">
      <van-button
       class="write-btn"
       size="small"
       round
       @click="isPostShow = true"
      >写评论</van-button>
    </div>

    <van-popup v-model="isPostShow" position="bottom">
      <comment-post
        :target="comment.com_id"
        @post-success="onPostSuccess"
      />
    </van-popup>
    <!-- 评论回复 -->
  </div>
</template>

<script>
import commentItem from './comment-item.vue'
import commentList from './comment-list.vue'
import commentPost from './comment-post.vue'

export default {
  name: 'CommentReply',
  components: {
    commentItem,
    commentList,
    commentPost
  },
  props: {
    comment: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      isPostShow: false,
      commentList: []
    }
  },
  methods: {
    onPostSuccess (data) {
      this.comment.reply_count++
      this.isPostShow = false
      this.commentList.unshift(data.new_obj)
    }
  }
}
</script>

<style scoped lang="less">
.scroll-wrap {
    position: fixed;
    top: 92px;
    left: 0;
    right: 0;
    bottom: 88px;
    overflow-y: auto;
}
.post-wrap {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    height: 88px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    border-top: 1px solid #d8d8d8;
    .write-btn {
        width: 60%;
    }
}
</style>
