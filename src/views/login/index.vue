<template>
  <div class="login-container">
    <!-- 导航栏 -->
    <van-nav-bar
     class="page-nav-bar"
     title="登录"
     >
     <van-icon
      slot="left"
      name="cross"
      @click="$router.back()"
       />
    </van-nav-bar>
    <!-- 导航栏 -->

    <!-- 登录表单 -->
    <van-form ref="loginForm" @submit="onSubmit">
      <van-field
        v-model="user.mobile"
        name="mobile"
        placeholder="请输入手机号"
        :rules="userFromRules.mobile"
        type="number"
        maxlength="11"
      >
      <i slot="left-icon" class="iconfont icon-shouji"></i>
      </van-field>
      <van-field
        v-model="user.code"
        name="code"
        placeholder="请输入验证码"
        :rules="userFromRules.code"
      >
      <i slot="left-icon" class="iconfont icon-suo"></i>
      <template #button>
      <van-count-down
       v-if="isCountDownShow"
       :time="60000"
       format="ss s"
       @finish="isCountDownShow = false"
       />
      <van-button
      v-else
       class="send-sms-btn"
       native-type="button"
       round size="small"
       type="default"
       @click="onSendSms"
         >获取验证码</van-button>
      </template>
      </van-field>
      <div class="login-btn-wrap">
        <van-button class="login-btn"  block type="info" native-type="submit">登录</van-button>
      </div>
    </van-form>
    <!-- 登录表单 -->
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user'

export default {
  name: 'LoginIndex',
  data () {
    return {
      user: {
        mobile: '',
        code: ''
      },
      userFromRules: {
        mobile: [
          { required: true, message: '手机号不能为空' },
          { pattern: /^1[3|5|7|8]\d{9}$/, message: '手机号格式错误' }
        ],
        code: [
          { required: true, message: '验证码不能为空' },
          { pattern: /^\d{6}$/, message: '验证码格式错误' }
        ]
      },
      isCountDownShow: false
    }
  },
  methods: {
    async onSubmit () {
      const user = this.user

      this.$toast.loading({
        message: '登录中...',
        forbidClick: true,
        duration: 0
      })

      try {
        const { data } = await login(user)
        this.$store.commit('setUser', data.data)
        this.$toast.success('登录成功')
        this.$router.back()
      } catch (err) {
        if (err.response.status === 400) {
          this.$toast.fail('手机号或验证码错误')
        } else {
          this.$toast.fail('登录失败，请稍后重试')
        }
      }
    },

    async onSendSms () {
      try {
        await this.$refs.loginForm.validate('mobile')
        // console.log('验证通过')
      } catch (err) {
        return console.log('验证失败', err)
      }
      this.isCountDownShow = true

      try {
        await sendSms(this.user.mobile)
        this.$toast('发送成功')
      } catch (err) {
        this.isCountDownShow = false
        if (err.response.status === 429) {
          this.$toast('发送太频繁了，请稍后再试')
        } else {
          this.$toast('发送失败，请稍后再试')
        }
      }
    }
  }
}
</script>

<style scoped lang="less">
.login-container{
  .iconfont{
    font-size: 37px;
  }

  .send-sms-btn{
    background-color: #ededed;
    width: 158px;
    height: 46px;
    line-height: 46px;
    font-size: 15px;
    color: #666;
  }

  .login-btn-wrap{
    padding: 53px 33px;
    .login-btn{
      background-color: #6db4fb;
      border: none;
    }
  }
}
</style>
