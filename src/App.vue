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
  <div class="app-container">
    <!-- Left Panel -->
    <div class="left-panel">
      <!-- Tab Headers -->
      <div class="tab-container">
        <input 
          type="radio" 
          name="tab" 
          id="tab1" 
          class="tab tab--1" 
          :checked="activeTab === 'notepad'"
          @change="activeTab = 'notepad'"
        />
        <label class="tab_label" for="tab1">Notepad Editor</label>

        <input 
          type="radio" 
          name="tab" 
          id="tab2" 
          class="tab tab--2" 
          :checked="activeTab === 'jsonstore'"
          @change="activeTab = 'jsonstore'"
        />
        <label class="tab_label" for="tab2">JSON Editor</label>

        <div class="indicator"></div>
      </div>
      
      <!-- Tab Content -->
      <div class="content-panel">
        <div v-if="activeTab === 'notepad'" class="panel-content">
          <Notepad v-model:lines="notepadLines" />
        </div>
        <div v-if="activeTab === 'jsonstore'" class="panel-content">
          <JsonPad v-model:lines="notepadLines" />
        </div>
      </div>
    </div>

    <!-- Right Panel -->
    <div class="right-panel">
      <div class="panel-header" style="text-align: center;">
        <h2>Node Diagram</h2>
      </div>
      <div class="content-panel">
        <Mappad :lines="notepadLines" />
      </div>
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
  background: #f8fafc;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

#app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

/* Main Layout */
.app-container {
  display: flex;
  height: 100vh;
  gap: 24px;
  padding: 24px;
  background: #f8fafc;
}

.left-panel,
.right-panel {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
}

.panel-header {
  margin-bottom: 16px;
}

.panel-header h2 {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
  color: #1e293b;
  letter-spacing: -0.025em;
}

.content-panel {
  flex: 1;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
  border: 1px solid #e2e8f0;
}

.panel-content {
  height: 100%;
  width: 100%;
}

/* Clean Modern Tab Styles */
.tab-container {
  position: relative;
  display: flex;
  flex-direction: row;
  align-items: center;
  background: #f1f5f9;
  border-radius: 10px;
  margin-bottom: 20px;
  border: 1px solid #e2e8f0;
}

.indicator {
  content: "";
  width: calc(50%);
  height: 36px;
  background: #3b82f6;
  position: absolute;
  z-index: 9;
  border-radius: 6px;
  transition: all 0.25s ease;
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.15);
}

.tab {
  width: 50%;
  height: 36px;
  position: absolute;
  z-index: 99;
  outline: none;
  opacity: 0;
  cursor: pointer;
}

.tab_label {
  width: 50%;
  height: 36px;
  position: relative;
  z-index: 999;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 0;
  font-size: 0.875rem;
  font-weight: 500;
  color: #64748b;
  cursor: pointer;
  transition: all 0.25s ease;
  border-radius: 6px;
}

.tab--1:checked ~ .indicator {
  left: 0px;
}

.tab--2:checked ~ .indicator {
  left: calc(50% + 0px);
}

.tab--1:checked + .tab_label,
.tab--2:checked + .tab_label {
  color: white;
  font-weight: 600;
}

.tab_label:hover {
  color: #475569;
}


</style>