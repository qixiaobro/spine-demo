<script setup>
import SimpleDemo from './components/SimpleDemo.vue'
import ThemeChangeDemo from './components/ThemeChangeDemo.vue';
import HttpSimpleDemo from './components/HttpSimpleDemo.vue';
import gpu from './components/gpu.vue';
import { ref } from 'vue'
const active = ref(1)

</script>

<template>
  <div>
    <div>
      <SimpleDemo v-if="active === 1" />
      <ThemeChangeDemo v-else-if="active === 2" />
      <HttpSimpleDemo v-else-if="active === 3" />
      <gpu v-else-if="active === 4" :id-prefix="1" :num="2" key="1"/>
      <gpu v-else-if="active === 5" :id-prefix="2" :num="10" key="2"/>
      <gpu v-else-if="active === 6" :id-prefix="3" :num="20" key="3"/>
    </div>
    <div style="margin-top:10px">
      <button :class="{ 'active': active === 1 }" @click="active = 1">Simple Demo</button>
      <button :class="{ 'active': active === 2 }" @click="active = 2" style="margin-left:10px">换肤 Demo</button>
      <button :class="{ 'active': active === 3 }" @click="active = 3" style="margin-left:10px">Http资源</button>
      <button :class="{ 'active': active === 4 }" @click="active = 4" style="margin-left:10px">性能测试2</button>
      <button :class="{ 'active': active === 5 }" @click="active = 5" style="margin-left:10px">性能测试10</button>
      <button :class="{ 'active': active === 6 }" @click="active = 6" style="margin-left:10px">性能测试20</button>

    </div>
  </div>
</template>

<style scoped>
.active {
  background-color: #747bff;
  color: #fff;
}
</style>