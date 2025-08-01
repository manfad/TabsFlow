<script lang="ts" setup>
import { ref } from 'vue'
import Notepad from './components/Notepad.vue';
import StoreJsonPad from './components/storeJsonPad.vue';

// Shared state for notepad lines
const notepadLines = ref([''])
</script>

<template>
  <div style="display: flex; gap: 20px; padding: 20px; height: 100vh;">
    <div style="flex: 1; display: flex; flex-direction: column; gap: 20px; min-height: 0;">
      <div style="flex: 1; display: flex; flex-direction: column; min-height: 0;">
        <h2 style="margin: 0 0 10px 0;">Notepad Editor</h2>
        <Notepad v-model:lines="notepadLines" style="flex: 1; min-height: 0;" />
      </div>
      <div style="flex: 1; display: flex; flex-direction: column; min-height: 0;">
        <h2 style="margin: 0 0 10px 0;">JSON Store</h2>
        <StoreJsonPad :lines="notepadLines" style="flex: 1; min-height: 0;" />
      </div>
    </div>
    <div style="flex: 1;">
      <!-- JsonPad will go here later -->
      <h2>JsonPad (Coming Soon)</h2>
    </div>
  </div>
</template>

<style>

</style>