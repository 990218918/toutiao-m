<template>
  <div class="my-container">
    <div v-if="user" class="header user-info">
      <div class="base-info">
        <div class="left">
          <van-image
            class="avatar"
            :src="userInfo.photo"
            round
            fit="cover"
          />
          <span class="name">{{ userInfo.name }}</span>
        </div>
        <div class="right">
          <van-button
           size="mini"
           round
           to="/user/profile"
           >编辑资料</van-button>
        </div>
      </div>
      <div class="data-stats">
        <div class="data-item">
          <span class="count">{{ userInfo.art_count}}</span>
          <span class="text">头条</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.follow_count}}</span>
          <span class="text">关注</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.fans_count }}</span>
          <span class="text">粉丝</span>
        </div>
        <div class="data-item">
          <span class="count">{{ userInfo.like_count }}</span>
          <span class="text">获赞</span>
        </div>
      </div>
    </div>

    <div v-else class="header not-login">
      <div class="login-btn" @click="$router.push('/login')">
        <img class="mobile-img" src="~@/assets/denglu.png" alt="">
        <span class="text">登录 / 注册</span>
      </div>
    </div>

    <!-- 导航 -->
    <van-grid class="grid-nav" :column-num="2" clickable>
      <van-grid-item class="grid-item">
        <i slot="icon" class="iconfont icon-shoucang"></i>
        <span slot="text" class="text">收藏</span>
      </van-grid-item>
      <van-grid-item class="grid-item">
        <i slot="icon" class="iconfont icon-shijian_o"></i>
        <span slot="text" class="text">历史 </span>
      </van-grid-item>
    </van-grid>
    <van-cell title="消息通知" is-link />
    <van-cell title="小智同学" is-link />
    <van-cell
     v-if="user"
     title="退出登录"
     class="logout-cell"
     clickable
     @click="onLogout"
     />
    <!-- 导航 -->
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { getUserInfo } from '@/api/user'

export default {
  name: 'MyIndex',
  data () {
    return {
      userInfo: {}
    }
  },
  computed: {
    ...mapState(['user'])
  },
  created () {
    if (this.user) {
      this.loadUserInfo()
    }
  },
  methods: {
    onLogout () {
      this.$dialog.confirm({
        title: '确认退出？'
      })
        .then(() => {
          // on confirm
          this.$store.commit('setUser', null)
        })
        .catch(() => {
          // on cancel
        })
    },
    async loadUserInfo () {
      try {
        const { data } = await getUserInfo()
        // console.log(data)
        this.userInfo = data.data
      } catch (err) {
        this.$toast('获取失败，请稍后重试')
      }
    }
  }
}
</script>

<style scoped lang="less">
.my-container {
  .header {
    height: 361px;
    background: url("~@/assets/bgc.png");
    background-size: cover;
  }
  .not-login {
    display: flex;
    justify-content: center;
    align-items: center;
    .login-btn {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      .mobile-img {
        border-radius:50%;
        height: 132px;
        width: 132px;
        margin-bottom: 15px;
      }
      .text {
        font-size: 28px;
        color: #666;
      }
    }
  }
  .user-info {
    .base-info {
      height: 231px;
      padding: 76px 32px 23px;
      box-sizing: border-box;
      display: flex;
      justify-content: space-between;
      align-items: center;
      .left {
        display: flex;
        align-items: center;
        .avatar {
          width: 132px;
          height: 132px;
          border: 2px solid #fff;
          margin-right: 23px;
        }
        .name {
          font-size: 30px;
          color: #fff;
        }
      }
    }

    .data-stats {
      display: flex;
      .data-item {
        height: 130px;
        flex: 1;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        color: #fff;
        .count {
          font-size: 36px;
        }
        .text {
          font-size: 23px;
        }
      }
    }
  }
  .grid-nav {
    .grid-item {
      height: 141px;
      i.iconfont {
        font-size: 45px;
      }
      .icon-shoucang {
        color: #eb5253;
      }
      .icon-shijian_o {
        color: #ff9d1d;
      }
      span.text {
        font-size: 28px;
      }
    }
  }
  .logout-cell {
    text-align: center;
    color: #eb5253;
    margin-top: 10px;
  }
}
</style>
