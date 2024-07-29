<script setup lang="ts">
import TheHighlighter from './components/TheHighlighter.vue';


</script>

<template>
  <header>
    <h1>The Highlighter</h1>
  </header>
  <main>
    <TheHighlighter />
  </main>
</template>

<style>
header {
  text-align: center;
}

main {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 2em 20em;
}
</style>
