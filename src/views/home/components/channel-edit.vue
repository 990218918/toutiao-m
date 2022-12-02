<template>
  <div class="channel-edit">
    <van-cell :border="false">
        <div slot="title" class="title-text">我的频道</div>
        <van-button
         class="edit-btn"
         type="danger"
         plain round
         size="mini"
         @click="isEdit = !isEdit"
         >{{ isEdit? '完成' : '编辑' }}</van-button>
    </van-cell>

    <van-grid class="my-grid" :gutter="10">
        <van-grid-item
         class="grid-item"
         v-for="(channel, index) in myChannels"
        :key="index"
        @click="onMyChannelClick(channel, index)"
        >
        <van-icon
        v-show="isEdit && !fiexdChannels.includes(channel.id)"
         slot="icon"
         name="clear"
        ></van-icon>
        <span
         class="text"
         :class="{ active: index === active }"
         slot="text"
         >{{channel.name}}</span>
    </van-grid-item>
    </van-grid>

    <van-cell :border="false">
        <div slot="title" class="title-text">频道推荐</div>
    </van-cell>

    <van-grid class="recommend-grid" :gutter="10">
        <van-grid-item
         class="grid-item"
         v-for="(channel, index) in recommendChannels"
         :key="index"
         icon="plus"
         :text="channel.name"
         @click="onAddChannel(channel)"
        />
    </van-grid>
  </div>
</template>

<script>
import {
  getAllChannels,
  addUserChannel,
  deleteUserChannel
} from '@/api/channel'
import { mapState } from 'vuex'
import { setItem } from '@/utils/storage'

export default {
  name: 'ChannelEdit',
  props: {
    myChannels: {
      type: Array,
      required: true
    },
    active: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      allChannels: [],
      isEdit: false,
      fiexdChannels: [0]
    }
  },
  computed: {
    ...mapState(['user']),
    recommendChannels () {
      const channels = []
      this.allChannels.forEach(channel => {
        const ret = this.myChannels.find(myChannel => {
          return myChannel.id === channel.id
        })
        if (!ret) {
          channels.push(channel)
        }
      })
      return channels
    }
  },
  created () {
    this.loadAllChannels()
  },
  methods: {
    async loadAllChannels () {
      try {
        const { data } = await getAllChannels()
        this.allChannels = data.data.channels
      } catch (err) {
        this.$toast('数据获取失败')
      }
    },
    async onAddChannel (channel) {
      this.myChannels.push(channel)
      if (this.user) {
        try {
          await addUserChannel({
            id: channel.id,
            seq: this.myChannels.length
          })
        } catch (err) {
          this.$toast('保存失败，请稍后重试！')
        }
      } else {
        setItem('TOUTIAO_CHANNELS', this.myChannels)
      }
    },
    onMyChannelClick (channel, index) {
      if (this.isEdit) {
        if (this.fiexdChannels.includes(channel.id)) {
          return
        }
        if (index <= this.active) {
          this.$emit('update-active', this.active - 1, true)
        }
        this.deleteChannel(channel)
        this.myChannels.splice(index, 1)
      } else {
        this.$emit('update-active', index, false)
      }
    },
    async deleteChannel (channel) {
      try {
        if (this.user) {
          await deleteUserChannel(channel.id)
        } else {
          setItem('TOUTIAO_CHANNEL', this.myChannels)
        }
      } catch (err) {
        this.$toast('操作失败，请稍后重试')
      }
    }
  }
}
</script>

<style scpoed lang="less">
.channel-edit {
    padding: 85px 0;
    .title-text {
        font-size: 32px;
        color: #333;
    }
    .edit-btn {
        width: 104px;
        height: 48px;
        font-size: 26px;
        color: #f85959;
        border: 1px solid #f85959;
    }
    .grid-item {
        width: 160px;
        height: 86px;
        .van-grid-item__content {
            white-space: nowrap;
            background-color: #f4f5f6;
            .van-grid-item__text, .text{
                font-size: 28px;
                color: #222;
                margin-top: 0;
            }
            .active {
                color: red;
            }
            .van-grid-item__icon-wrapper {
                position: unset;
            }
        }
    }
    .my-grid {
        .grid-item {
            .van-icon-clear {
                position: absolute;
                right: -10px;
                top: -10px;
                font-size: 30px;
                color: #cacaca;
                z-index: 2;
            }
        }
    }
    .recommend-grid {
        .grid-item {
            .van-grid-item__content {
                flex-direction: row;
                .van-icon-plus {
                    font-size: 28px;
                    margin-right: 6px;
                }
            }
        }
    }
}
</style>
