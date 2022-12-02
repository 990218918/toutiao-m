<template>
  <div class="search-history">
    <van-cell title="搜索历史">
      <div v-if="isDeleteShow">
        <span @click="$emit('clear-search-histories', [])">全部删除</span>
        <span @click="isDeleteShow = false">完成</span>
      </div>
      <van-icon
       v-else
       name="delete"
       @click="isDeleteShow = true"
       />
    </van-cell>
    <van-cell
     :title="item"
     v-for="(item, index) in searchHistories"
     :key="index"
     @click="onSearchItemClick(item, index)"
     >
        <van-icon
         name="close"
         v-if="isDeleteShow"
         />
    </van-cell>
  </div>
</template>

<script>
export default {
  name: 'SearchHistory',
  props: {
    searchHistories: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      isDeleteShow: false
    }
  },
  methods: {
    onSearchItemClick (item, index) {
      if (this.isDeleteShow) {
        this.searchHistories.splice(index, 1)
      } else {
        this.$emit('search', item)
      }
    }
  }
}
</script>

<style scoped lang="less">
.search-history {
  .van-cell__value span{
    margin-left: 10px;
  }
}

</style>
