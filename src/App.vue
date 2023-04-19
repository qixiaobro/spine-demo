<script setup>
import SimpleDemo from './components/SimpleDemo.vue'
import ThemeChangeDemo from './components/ThemeChangeDemo.vue';
import { ref } from 'vue'
const active = ref(1)

</script>

<template>
  <div>
    <div>
      <SimpleDemo v-if="active === 1" />
      <ThemeChangeDemo v-else />
    </div>
    <div style="margin-top:10px">
      <button @click="active = 1">Simple Demo</button>
      <button @click="active = 2" style="margin-left:10px">换肤 Demo</button>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>