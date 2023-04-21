<script setup>
import { onMounted, ref } from 'vue';
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

// const count = ref(1)
// onMounted(() => {
//   setInterval(() => {
//     count.value++
//   }, 3000)
// })
</script>

<template>
  <div v-for="i in num" :key="i">
    <SimpleDemo :id="`${idPrefix}${i}`" />
  </div>
</template>

<style lang="scss" scoped></style>