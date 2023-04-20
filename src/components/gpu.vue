<script setup>
import SimpleDemo from './SimpleDemo.vue'

defineProps({
  idPrefix: {
    type: String,
    default: '1'
  },
  num: {
    type: Number,
    default: 2
  }
})
</script>

<template>
<div>
  <SimpleDemo v-for="i in num" :key="i" :id="`${idPrefix}${i}`"/>
</div>
</template>

<style lang="scss" scoped>
</style>