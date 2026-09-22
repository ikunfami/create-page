<template>
  <view class="container">
    <text class="title">{{ title }}</text>
  </view>
</template>

<script>
export default {
  data() {
    return {
      title: '{{PAGE_TITLE}}',
    }
  },
}
</script>

<style scoped>
.container {
  padding: 24rpx;
}

.title {
  font-size: 32rpx;
}
</style>
