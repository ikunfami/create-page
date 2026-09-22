<template>
  <view class="container">
    <text class="title">{{ title }}</text>
  </view>
</template>

<script setup>
import { ref } from 'vue'

const title = ref('{{PAGE_TITLE}}')
</script>

<style scoped>
.container {
  padding: 24rpx;
}

.title {
  font-size: 32rpx;
}
</style>
