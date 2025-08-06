<script lang="ts" setup>
import { ref } from 'vue'
import Notepad from './components/Notepad.vue';
import JsonPad from './components/JsonPad.vue';
import Mappad from './components/MapPad.vue';

// Shared state for notepad lines
const notepadLines = ref([''])

// Tab state
const activeTab = ref<'notepad' | 'jsonstore'>('notepad')
</script>

<template>
  <div style="display: flex; gap: 20px; padding: 20px; height: 100vh; overflow: hidden;">
    <div style="flex: 1; display: flex; flex-direction: column; min-height: 0; min-width: 0; max-width: 50%;">
      <!-- Tab Headers -->
      <div class="tab-headers">
        <button 
          class="tab-button" 
          :class="{ active: activeTab === 'notepad' }"
          @click="activeTab = 'notepad'"
        >
          📝 Notepad Editor
        </button>
        <button 
          class="tab-button" 
          :class="{ active: activeTab === 'jsonstore' }"
          @click="activeTab = 'jsonstore'"
        >
          🗂️ JSON Store
        </button>
      </div>
      
      <!-- Tab Content -->
      <div class="tab-content">
        <div v-if="activeTab === 'notepad'" style="height: 100%; display: flex; flex-direction: column;">
          <Notepad v-model:lines="notepadLines" style="flex: 1; min-height: 0; min-width: 0;" />
        </div>
        <div v-if="activeTab === 'jsonstore'" style="height: 100%; display: flex; flex-direction: column;">
          <JsonPad :lines="notepadLines" style="flex: 1; min-height: 0; min-width: 0;" />
        </div>
      </div>
    </div>
    <div style="flex: 1; display: flex; flex-direction: column; min-height: 0; min-width: 0; max-width: 50%;">
      <h2 style="margin: 0 0 10px 0;">Diagram</h2>
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

/* Tab Styles */
.tab-headers {
  display: flex;
  border-bottom: 2px solid #e1e5e9;
  margin-bottom: 10px;
  background: #f8f9fa;
  border-radius: 8px 8px 0 0;
}

.tab-button {
  flex: 1;
  padding: 12px 16px;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: #6c757d;
  transition: all 0.2s ease;
  border-radius: 8px 8px 0 0;
  position: relative;
}

.tab-button:hover {
  background: rgba(0, 123, 255, 0.1);
  color: #007bff;
}

.tab-button.active {
  background: white;
  color: #007bff;
  font-weight: 600;
  border-bottom: 2px solid #007bff;
  margin-bottom: -2px;
}

.tab-content {
  flex: 1;
  min-height: 0;
  background: white;
  border-radius: 0 0 8px 8px;
  overflow: hidden;
}
</style>