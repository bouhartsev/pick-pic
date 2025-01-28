<script setup lang="ts">
import MainCanvas from '../components/MainCanvas.vue';
import SideTools from '../components/SideTools.vue';
</script>

<template>
  <aside>
    <SideTools />
  </aside>
  <main>
    <MainCanvas />
  </main>
</template>

<style scoped>
aside,
main {
  height: calc(100vh - 2rem * 2 - 10vh);
}

aside {
  float: right;
  width: 25%;
  margin-left: 5%;
}
main {
  width: 70%;
}
</style>