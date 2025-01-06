<script setup lang="ts">
// import HelloWorld from './components/HelloWorld.vue'
import ProseMirror from './components/ProseMirror.vue';
import TiptapEditor from './components/TiptapEditor.vue';
</script>

<template>
  <h2>By ProseMirror</h2>
  <ProseMirror />
  <h2>By Tiptap</h2>
  <TiptapEditor />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
