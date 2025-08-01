<script lang="ts" setup>
import { ref } from 'vue'
import Notepad from './components/Notepad.vue';
import JsonPad from './components/JsonPad.vue';
import Mappad from './components/Mappad.vue';

// Shared state for notepad lines
const notepadLines = ref([''])
</script>

<template>
  <div style="display: flex; gap: 20px; padding: 20px; height: 100vh; overflow: hidden;">
    <div style="flex: 1; display: flex; flex-direction: column; gap: 20px; min-height: 0; min-width: 0; max-width: 50%;">
      <div style="flex: 1; display: flex; flex-direction: column; min-height: 0; min-width: 0;">
        <h2 style="margin: 0 0 10px 0;">Notepad Editor</h2>
        <Notepad v-model:lines="notepadLines" style="flex: 1; min-height: 0; min-width: 0;" />
      </div>
      <div style="flex: 1; display: flex; flex-direction: column; min-height: 0; min-width: 0;">
        <h2 style="margin: 0 0 10px 0;">JSON Store</h2>
        <JsonPad :lines="notepadLines" style="flex: 1; min-height: 0; min-width: 0;" />
      </div>
    </div>
    <div style="flex: 1; display: flex; flex-direction: column; min-height: 0; min-width: 0; max-width: 50%;">
      <h2 style="margin: 0 0 10px 0;">MapPad</h2>
      <Mappad :lines="notepadLines" style="flex: 1; min-height: 0; min-width: 0;" />
    </div>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>