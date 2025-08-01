<script lang="ts" setup>
import { computed } from 'vue'

const props = defineProps<{
  lines: string[]
}>()

interface ParsedLine {
  level: number
  text: string
  originalIndex: number
}

const parseLines = computed((): ParsedLine[] => {
  return props.lines.map((line, index) => {
    let level = 0
    let text = line
    
    // Count leading tabs
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
    
    return {
      level,
      text: text.trim(),
      originalIndex: index
    }
  }).filter(line => line.text !== '') // Remove empty lines
})

const buildHierarchy = computed(() => {
  const lines = parseLines.value
  const result: any = {}
  
  if (lines.length === 0) return result
  
  // Use a stack-based approach to handle same strings at different levels
  const stack: { container: any; level: number }[] = [{ container: result, level: -1 }]
  const processedIndices = new Set<number>()
  
  for (let i = 0; i < lines.length; i++) {
    // Skip if this line was already processed as a child
    if (processedIndices.has(i)) continue
    
    const currentLine = lines[i]
    const { level, text } = currentLine
    
    // Pop stack until we find the right parent level
    while (stack.length > 1 && stack[stack.length - 1].level >= level) {
      stack.pop()
    }
    
    const currentContainer = stack[stack.length - 1].container
    
    // Create unique key to handle duplicate strings at different levels
    let key = text
    let counter = 1
    while (currentContainer.hasOwnProperty(key)) {
      key = `${text}_${counter}`
      counter++
    }
    
    // Look ahead to determine if this will be an object or array
    const childLines: ParsedLine[] = []
    const childIndices: number[] = []
    for (let j = i + 1; j < lines.length && lines[j].level > level; j++) {
      childLines.push(lines[j])
      childIndices.push(j)
    }
    
    if (childLines.length === 0) {
      // No children - empty object
      currentContainer[key] = {}
    } else {
      // Check immediate children
      const immediateChildren = childLines.filter(line => line.level === level + 1)
      const immediateChildIndices = childIndices.filter((_, idx) => childLines[idx].level === level + 1)
      const hasGrandchildren = childLines.some(line => line.level > level + 1)
      
      if (immediateChildren.length > 0 && hasGrandchildren) {
        // Has both immediate children and grandchildren - create object
        currentContainer[key] = {}
        stack.push({ container: currentContainer[key], level })
      } else if (immediateChildren.length > 0) {
        // Only immediate children - create array and mark them as processed
        currentContainer[key] = immediateChildren.map(line => line.text)
        immediateChildIndices.forEach(idx => processedIndices.add(idx))
      } else {
        // Only deeper descendants - collect all as array and mark as processed
        currentContainer[key] = childLines.map(line => line.text)
        childIndices.forEach(idx => processedIndices.add(idx))
      }
    }
  }
  
  return result
})

const formattedJson = computed(() => {
  return JSON.stringify(buildHierarchy.value, null, 2)
})
</script>

<template>
  <div class="json-pad">
    <div class="header">
      <h3>Hierarchical JSON Output</h3>
      <div class="stats">
        <span>{{ parseLines.length }} items</span>
        <span>{{ Object.keys(buildHierarchy).length }} top-level</span>
      </div>
    </div>
    
    <div class="json-display">
      <pre class="json-content">{{ formattedJson }}</pre>
    </div>
  </div>
</template>

<style scoped>
.json-pad {
  font-family: 'Courier New', monospace;
  padding: 16px;
  background-color: #f0f8ff;
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
  border-bottom: 1px solid #b3d9ff;
}

.header h3 {
  margin: 0;
  color: #2c5aa0;
}

.stats {
  font-size: 12px;
  color: #5a7db8;
}

.stats span {
  margin-left: 12px;
  padding: 2px 6px;
  background-color: #e6f2ff;
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
  border: 1px solid #b3d9ff;
  border-radius: 4px;
  padding: 12px;
  font-size: 12px;
  overflow-y: auto;
  white-space: pre-wrap;
  color: #2c5aa0;
  flex: 1;
  min-height: 0;
}
</style>