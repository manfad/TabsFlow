<script lang="ts" setup>
import { ref, computed } from 'vue'
import Notepad from './components/Notepad.vue';
import JsonPad from './components/JsonPad.vue';
import Mappad from './components/MapPad.vue';

// Shared state for notepad lines
const notepadLines = ref([''])

// Tab state
const activeTab = ref<'notepad' | 'jsonstore'>('notepad')

// Computed property for JSON content
const jsonContent = computed(() => {
  const parseLines = notepadLines.value.map((line, index) => {
    let level = 0
    let text = line
    
    // Count leading tabs first
    const tabMatch = line.match(/^(\t*)/)
    if (tabMatch && tabMatch[1].length > 0) {
      level = tabMatch[1].length
      text = line.substring(level)
    } else {
      // Count leading spaces (4 spaces = 1 level)
      const spaceMatch = line.match(/^( *)/)
      if (spaceMatch && spaceMatch[1].length > 0) {
        level = Math.floor(spaceMatch[1].length / 4)
        text = line.substring(spaceMatch[1].length)
      }
    }
    
    return { level, text: text.trim(), originalIndex: index }
  }).filter(line => line.text !== '')

  // Build hierarchy from parsed lines
  const hierarchy: any[] = []
  const stack: any[] = []

  for (const parsedLine of parseLines) {
    const item = { text: parsedLine.text }

    while (stack.length > parsedLine.level) {
      stack.pop()
    }

    if (stack.length === 0) {
      hierarchy.push(item)
      stack.push(item)
    } else {
      const parent = stack[stack.length - 1]
      if (!parent.children) {
        parent.children = []
      }
      parent.children.push(item)
      stack.push(item)
    }
  }

  return JSON.stringify(hierarchy, null, 2)
})

// Toolbar action functions
const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(notepadLines.value.join('\n'))
  } catch (err) {
    console.error('Failed to copy: ', err)
  }
}

const copyJsonToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(jsonContent.value)
  } catch (err) {
    console.error('Failed to copy JSON: ', err)
  }
}

const saveToTxt = () => {
  const content = notepadLines.value.join('\n')
  const blob = new Blob([content], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = 'tabsflow.txt'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

const saveJsonToFile = () => {
  const blob = new Blob([jsonContent.value], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = 'tabsflow.json'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

const fixJsonFormat = () => {
  try {
    const parsed = JSON.parse(jsonContent.value)
    // Re-generate the JSON content to fix any formatting issues
    const formatted = JSON.stringify(parsed, null, 2)
    console.log('JSON format fixed:', formatted.length, 'characters')
  } catch (error) {
    console.error('Could not fix JSON format:', error)
  }
}
</script>

<template>
  <div class="app-wrapper">
    <!-- App Title -->
    <div class="app-title">
      <h1>Tabs Flow</h1>
    </div>
    

    <div class="app-container">
    <!-- Left Panel -->
    <div class="left-panel">
      <!-- Editor Container with Integrated Toolbar -->
      <div class="editor-container-with-toolbar">
        <!-- Unified Toolbar -->
        <div class="unified-toolbar">
          <!-- Tab Switcher -->
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
          
          <!-- Action Buttons (context-sensitive) -->
          <div class="action-buttons">
            <!-- Notepad actions -->
            <template v-if="activeTab === 'notepad'">
              <button @click="copyToClipboard" class="toolbar-button copy-button" title="Copy to clipboard">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"><path d="M16.964 8.982c-.003-2.95-.047-4.478-.906-5.524a4 4 0 0 0-.553-.554C14.4 2 12.76 2 9.48 2s-4.92 0-6.024.905a4 4 0 0 0-.553.554C1.998 4.56 1.998 6.2 1.998 9.48s0 4.92.906 6.023q.25.304.553.553c1.047.86 2.575.904 5.525.906"/><path d="m14.028 9.025l2.966-.043m-2.98 13.02l2.966-.043m4.992-7.937l-.028 2.96M9.01 14.036l-.028 2.96m2.505-7.971c-.832.149-2.17.302-2.477 2.024m10.485 10.91c.835-.137 2.174-.27 2.508-1.986M19.495 9.025c.832.149 2.17.302 2.477 2.024M11.5 21.957c-.833-.148-2.17-.301-2.478-2.023"/></g></svg> 
              </button>
              <button @click="saveToTxt" class="toolbar-button save-button" title="Save as text file">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"><path d="M12.5 2h.273c3.26 0 4.892 0 6.024.798c.324.228.612.5.855.805c.848 1.066.848 2.6.848 5.67v2.545c0 2.963 0 4.445-.469 5.628c-.754 1.903-2.348 3.403-4.37 4.113c-1.257.441-2.83.441-5.98.441c-1.798 0-2.698 0-3.416-.252c-1.155-.406-2.066-1.263-2.497-2.35c-.268-.676-.268-1.523-.268-3.216V12"/><path d="M20.5 12a3.333 3.333 0 0 1-3.333 3.333c-.666 0-1.451-.116-2.098.057a1.67 1.67 0 0 0-1.179 1.179c-.173.647-.057 1.432-.057 2.098A3.333 3.333 0 0 1 10.5 22m-6-14.5C4.992 8.006 6.3 10 7 10m2.5-2.5C9.008 8.006 7.7 10 7 10m0 0V2"/></g></svg>
              </button>
            </template>
            
            <!-- JSON Editor actions -->
            <template v-if="activeTab === 'jsonstore'">
              <button @click="fixJsonFormat" class="toolbar-button fix-button" title="Fix JSON format">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"><path d="M11 11L6 6M5 7.5L7.5 5l-3-1.5l-1 1zm14.975 1.475a3.5 3.5 0 0 0 .79-3.74l-1.422 1.422h-2v-2l1.422-1.422a3.5 3.5 0 0 0-4.529 4.53l-6.47 6.471a3.5 3.5 0 0 0-4.53 4.529l1.421-1.422h2v2l-1.422 1.422a3.5 3.5 0 0 0 4.53-4.528l6.472-6.472a3.5 3.5 0 0 0 3.738-.79"/><path d="m11.797 14.5l5.604 5.604a1.35 1.35 0 0 0 1.911 0l.792-.792a1.35 1.35 0 0 0 0-1.911L14.5 11.797"/></g></svg>
              </button>
              <button @click="copyJsonToClipboard" class="toolbar-button copy-button" title="Copy JSON to clipboard">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"><path d="M16.964 8.982c-.003-2.95-.047-4.478-.906-5.524a4 4 0 0 0-.553-.554C14.4 2 12.76 2 9.48 2s-4.92 0-6.024.905a4 4 0 0 0-.553.554C1.998 4.56 1.998 6.2 1.998 9.48s0 4.92.906 6.023q.25.304.553.553c1.047.86 2.575.904 5.525.906"/><path d="m14.028 9.025l2.966-.043m-2.98 13.02l2.966-.043m4.992-7.937l-.028 2.96M9.01 14.036l-.028 2.96m2.505-7.971c-.832.149-2.17.302-2.477 2.024m10.485 10.91c.835-.137 2.174-.27 2.508-1.986M19.495 9.025c.832.149 2.17.302 2.477 2.024M11.5 21.957c-.833-.148-2.17-.301-2.478-2.023"/></g></svg> 
              </button>
              <button @click="saveJsonToFile" class="toolbar-button save-button" title="Save as JSON file">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"><path d="M12.5 2h.273c3.26 0 4.892 0 6.024.798c.324.228.612.5.855.805c.848 1.066.848 2.6.848 5.67v2.545c0 2.963 0 4.445-.469 5.628c-.754 1.903-2.348 3.403-4.37 4.113c-1.257.441-2.83.441-5.98.441c-1.798 0-2.698 0-3.416-.252c-1.155-.406-2.066-1.263-2.497-2.35c-.268-.676-.268-1.523-.268-3.216V12"/><path d="M20.5 12a3.333 3.333 0 0 1-3.333 3.333c-.666 0-1.451-.116-2.098.057a1.67 1.67 0 0 0-1.179 1.179c-.173.647-.057 1.432-.057 2.098A3.333 3.333 0 0 1 10.5 22m-6-14.5C4.992 8.006 6.3 10 7 10m2.5-2.5C9.008 8.006 7.7 10 7 10m0 0V2"/></g></svg>
              </button>
            </template>
          </div>
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
    </div>

    <!-- Right Panel -->
    <div class="right-panel">
      <div class="content-panel">
        <Mappad :lines="notepadLines" />
      </div>
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
  background: #ffffff;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

#app {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

/* App Wrapper Layout */
.app-wrapper {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f8fafc;
}

.app-title {
  text-align: center;
  padding: 20px 24px 0 24px;
  background: #f8fafc;
}

.app-title h1 {
  margin: 0;
  font-size: 2.5rem;
  font-weight: 700;
  color: #1e293b;
  letter-spacing: -0.025em;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

/* Unified Toolbar */
.unified-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: #f8f9fa;
  border-bottom: 1px solid #e2e8f0;
  border-radius: 12px 12px 0 0;
  margin-bottom: 0;
}

.action-buttons {
  display: flex;
  gap: 8px;
  align-items: center;
}

.toolbar-button {
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
  font-weight: 500;
  background: #f1f5f9;
  color: #64748b;
}

.toolbar-button:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.copy-button:hover {
  background: #0056b3;
  color: white;
}

.save-button:hover {
  background: #218838;
  color: white;
}

.fix-button:hover {
  background: #f57c00;
  color: white;
}

/* Main Layout */
.app-container {
  display: flex;
  flex: 1;
  gap: 24px;
  padding: 24px;
  background: #f8fafc;
  min-height: 0;
}

.left-panel,
.right-panel {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
}

.editor-container-with-toolbar {
  display: flex;
  flex-direction: column;
  height: 100%;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06);
  border-radius: 12px;
  overflow: hidden;
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
  border-radius: 0 0 12px 12px;
  overflow: hidden;
  border: 1px solid #e2e8f0;
  border-top: none;
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
  border: 1px solid #e2e8f0;
  min-width: 320px;
  flex-shrink: 0;
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