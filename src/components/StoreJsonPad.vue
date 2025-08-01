<script lang="ts" setup>
import { computed } from 'vue'

const props = defineProps<{
  lines: string[]
}>()

interface LineData {
  line: number
  level: number
  text: string
}

const analyzeLines = computed((): LineData[] => {
  return props.lines.map((lineContent, index) => {
    let level = 0
    let text = lineContent
    
    // Count leading tabs
    const tabMatch = lineContent.match(/^(\t*)/)
    if (tabMatch && tabMatch[1].length > 0) {
      level = tabMatch[1].length
      text = lineContent.substring(level)
    } else {
      // Count leading spaces (4 spaces = 1 level)
      const spaceMatch = lineContent.match(/^( *)/)
      if (spaceMatch && spaceMatch[1].length > 0) {
        level = Math.floor(spaceMatch[1].length / 4)
        text = lineContent.substring(spaceMatch[1].length)
      }
    }
    
    return {
      line: index,
      level: level,
      text: text
    }
  })
})

const formattedJson = computed(() => {
  return JSON.stringify(analyzeLines.value, null, 2)
})
</script>

<template>
  <div class="store-json-pad">
    <div class="header">
      <h3>Notepad JSON Structure</h3>
      <div class="stats">
        <span>{{ props.lines.length }} lines</span>
        <span>{{ analyzeLines.filter(l => l.level > 0).length }} indented</span>
      </div>
    </div>
    
    <div class="json-display">
      <pre class="json-content">{{ formattedJson }}</pre>
    </div>
    

  </div>
</template>

<style scoped>
.store-json-pad {
  font-family: 'Courier New', monospace;
  padding: 16px;
  background-color: #f8f9fa;
  border-radius: 8px;
  height: 100%;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 8px;
  border-bottom: 1px solid #dee2e6;
}

.header h3 {
  margin: 0;
  color: #495057;
}

.stats {
  font-size: 12px;
  color: #6c757d;
}

.stats span {
  margin-left: 12px;
  padding: 2px 6px;
  background-color: #e9ecef;
  border-radius: 4px;
}

.json-display {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.json-content {
  background-color: #ffffff;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  padding: 12px;
  font-size: 12px;
  overflow-y: auto;
  white-space: pre-wrap;
  color: #495057;
  flex: 1;
  min-height: 0;
}


</style>