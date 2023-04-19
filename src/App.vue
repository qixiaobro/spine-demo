<script setup>
import SimpleDemo from './components/SimpleDemo.vue'
import ThemeChangeDemo from './components/ThemeChangeDemo.vue';
import HttpSimpleDemo from './components/HttpSimpleDemo.vue';
import { ref } from 'vue'
const active = ref(3)

</script>

<template>
  <div>
    <div>
      <SimpleDemo v-if="active === 1" />
      <ThemeChangeDemo v-else-if="active === 2" />
      <HttpSimpleDemo v-else-if="active === 3" />
    </div>
    <div style="margin-top:10px">
      <button @click="active = 1">Simple Demo</button>
      <button @click="active = 2" style="margin-left:10px">换肤 Demo</button>
      <button @click="active = 3" style="margin-left:10px">Http资源</button>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>