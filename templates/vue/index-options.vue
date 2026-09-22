<template>
  <div class="{{PAGE_NAME}}-page">
    <h2 class="page-title">{{ title }}</h2>
  </div>
</template>

<script>
export default {
  name: '{{PAGE_NAME_PASCAL}}',
  data() {
    return {
      title: '{{PAGE_TITLE}}',
    }
  },
}
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
