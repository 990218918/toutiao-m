<template>
  <div class="user-profile">
    <!-- 导航 -->
    <van-nav-bar
      class="page-nav-bar"
      title="个人信息"
      left-arrow
      @click-left="$router.back()"
    />
    <!-- 导航 -->

    <input
     type="file"
     hidden
     ref="file"
     @change="onFileChange"
    >

    <!-- 个人信息 -->
    <van-cell
     title="头像"
     is-link
     @click="$refs.file.click()"
    >
      <van-image
        class="avatar"
        fit="cover"
        round
        :src="user.photo"
      />
    </van-cell>
    <van-cell
     title="昵称"
     :value="user.name"
     is-link
     @click="isUpdateNameShow = true"
    />
    <van-cell
     title="性别"
     :value="user.gender === 0 ? '男' : '女'"
     is-link
     @click="isUpdateGenderShow = true"
    />
    <van-cell
     title="生日"
     :value="user.birthday"
     is-link
     @click="isUpdateBirthdayShow = true"
    />
    <!-- 个人信息 -->

    <!-- 编辑头像 -->
    <van-popup
     v-model="isUpdatePhotoShow"
     position="bottom"
     style="height: 100%"
    >
      <update-photo
        v-if="isUpdatePhotoShow"
        :img="img"
        @close="isUpdatePhotoShow = false"
        @update-photo="user.photo = $event"
      />
    </van-popup>
    <!-- 编辑头像 -->

    <!-- 编辑昵称 -->
    <van-popup
     v-model="isUpdateNameShow"
     style="height: 100%"
     position="bottom"
    >
      <update-name
       v-if="isUpdateNameShow"
       v-model="user.name"
       @close="isUpdateNameShow = false"
      />
    </van-popup>
    <!-- 编辑昵称 -->

    <!-- 编辑性别 -->
    <van-popup
     v-model="isUpdateGenderShow"
     position="bottom"
    >
      <update-gender
        v-if="isUpdateGenderShow"
        v-model="user.gender"
        @close="isUpdateGenderShow = false"
      />
    </van-popup>
    <!-- 编辑性别 -->

    <!-- 编辑生日 -->
    <van-popup
     v-model="isUpdateBirthdayShow"
     position="bottom"
    >
    <update-birthday
      v-if="isUpdateBirthdayShow"
      v-model="user.birthday"
      @close="isUpdateBirthdayShow = false"
    />
    </van-popup>
    <!-- 编辑生日 -->
  </div>
</template>

<script>
import { getUserProfile } from '@/api/user'
import UpdateName from './components/update-name.vue'
import UpdateGender from './components/update-gender.vue'
import UpdateBirthday from './components/update-birthday.vue'
import UpdatePhoto from './components/update-photo.vue'

export default {
  name: 'UserProfile',
  components: {
    UpdateName,
    UpdateGender,
    UpdateBirthday,
    UpdatePhoto
  },
  data () {
    return {
      user: {},
      isUpdateNameShow: false,
      isUpdateGenderShow: false,
      isUpdateBirthdayShow: false,
      isUpdatePhotoShow: false,
      img: null
    }
  },
  created () {
    this.loadUserProfile()
  },
  methods: {
    async loadUserProfile () {
      try {
        const { data } = await getUserProfile()
        this.user = data.data
      } catch (err) {
        this.$toast('获取数据失败')
      }
    },
    onFileChange () {
      const file = this.$refs.file.files[0]
      this.img = window.URL.createObjectURL(file)
      this.isUpdatePhotoShow = true
      this.$refs.file.value = ''
    }
  }
}
</script>

<style scoped lang="less">
.user-profile {
    .avatar {
        height: 60px;
        width: 60px;
    }
    .van-popup {
        background-color: #f5f7f9;
    }
}
</style>
