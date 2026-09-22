<template>
  <div class="{{PAGE_NAME}}-page">
    <h2 class="page-title">{{ title }}</h2>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const title = ref('{{PAGE_TITLE}}')
</script>

<style scoped>
.{{PAGE_NAME}}-page {
  padding: 16px;
}

.page-title {
  margin: 0;
  font-size: 18px;
}
</style>
