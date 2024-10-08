<script setup lang="ts">
import { ref } from 'vue'
import ChatInterface from './components/ChatInterface.vue'

const title = ref('Data Model Builder')
</script>

<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <ChatInterface />
  </div>
</template>

<style>
:root {
  --bg-color: #1a1a1a;
  --text-color: #e0e0e0;
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --accent-color: #e74c3c;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
  font-family: Arial, sans-serif;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: var(--primary-color);
}
</style>