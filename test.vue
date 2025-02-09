<template>
  <div>
    <p>WebSocket Status: {{ status }}</p>
    <div>
      <h2>Messages</h2>
      <ul>
        <li v-for="(message, index) in messages" :key="index">
          {{ message }}
        </li>
      </ul>
    </div>
    <button @click="isOpen = true">Open Drawer</button>

    <utilitycomponentsDrawer v-model="isOpen" :height="320" position="bottom" marginLeft="0%" width="100%%">
      <h2>Drawer Content</h2>
      <p>Drag the handle or click outside to close.</p>
    </utilitycomponentsDrawer>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useWebSocket } from '@vueuse/core'
const isOpen = ref(false);
 
// WebSocket URL
  



definePageMeta({
  layout: 'blank',
})
</script>

<style scoped>
/* Add simple styling */
input {
  margin-right: 8px;
  padding: 4px;
}

button {
  margin: 4px;
  padding: 6px 12px;
}
</style>
