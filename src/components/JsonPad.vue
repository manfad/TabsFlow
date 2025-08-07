<script lang="ts" setup>
import { computed, ref } from 'vue'
import { VueMonacoEditor } from '@guolao/vue-monaco-editor'

const props = defineProps<{
  lines: string[]
}>()

const emit = defineEmits<{
  'update:lines': [lines: string[]]
  'fix-json': []
}>()

interface ParsedLine {
  level: number
  text: string
  originalIndex: number
}

// Editor state
const jsonContent = ref('')
const isUserEditing = ref(false)
let editingTimer: number | null = null

// Initialize JSON content when component mounts or lines change
const initializeJsonContent = () => {
  // Don't update if user is actively editing
  if (isUserEditing.value) return
  
  const hierarchy = buildHierarchy.value
  const newJsonContent = JSON.stringify(hierarchy, null, 2)
  
  // Only update if content actually changed to prevent cursor jumps
  if (jsonContent.value !== newJsonContent) {
    jsonContent.value = newJsonContent
  }
}

// Parse lines to build hierarchy (existing logic)
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

// Convert JSON back to indented text
const convertJsonToLines = (jsonObj: any, level: number = 0): string[] => {
  const lines: string[] = []
  
  for (const [key, value] of Object.entries(jsonObj)) {
    // Add the key with proper indentation
    const indent = '\t'.repeat(level)
    lines.push(`${indent}${key}`)
    
    if (Array.isArray(value)) {
      // Handle arrays - add each item as a child
      value.forEach((item) => {
        const childIndent = '\t'.repeat(level + 1)
        lines.push(`${childIndent}${item}`)
      })
    } else if (typeof value === 'object' && value !== null) {
      // Handle nested objects - recurse
      const childLines = convertJsonToLines(value, level + 1)
      lines.push(...childLines)
    }
    // If value is primitive (string, number, boolean), it's already handled as empty object
  }
  
  return lines
}

// Handle JSON editing and conversion with debouncing
let debounceTimer: number | null = null

const handleJsonChange = (value: string) => {
  jsonContent.value = value
  
  // Mark user as actively editing
  isUserEditing.value = true
  
  // Clear existing timers
  if (debounceTimer) {
    clearTimeout(debounceTimer)
  }
  if (editingTimer) {
    clearTimeout(editingTimer)
  }
  
  // Reset editing state after user stops typing
  editingTimer = setTimeout(() => {
    isUserEditing.value = false
  }, 2000) // 2 seconds after last keystroke
  
  // Debounce the conversion to avoid too frequent updates
  debounceTimer = setTimeout(() => {
    try {
      const parsedJson = JSON.parse(value)
      const newLines = convertJsonToLines(parsedJson)
      emit('update:lines', newLines)
    } catch (error) {
      // Silently ignore invalid JSON while typing
      // console.error('Invalid JSON:', error)
    }
  }, 500) // 500ms delay
}

// Fix JSON format by wrapping in braces and removing trailing commas
// Initialize when lines change
const initializeContent = () => {
  initializeJsonContent()
}

// Watch for changes in props.lines to update JSON content
import { watch } from 'vue'
watch(() => props.lines, initializeContent, { immediate: true })

// Monaco editor options
const editorOptions = {
  theme: 'vs',
  language: 'json',
  fontSize: 14,
  fontFamily: 'Courier New, monospace',
  lineNumbers: 'on' as const,
  minimap: { enabled: false },
  scrollBeyondLastLine: false,
  wordWrap: 'off' as const,
  automaticLayout: true,
  tabSize: 4,
  insertSpaces: false,
  detectIndentation: false,
  renderWhitespace: 'selection' as const,
  renderControlCharacters: false,
  folding: false,
  rulers: [],
  overviewRulerBorder: false,
  hideCursorInOverviewRuler: true,
  scrollbar: {
    vertical: 'auto' as const,
    horizontal: 'auto' as const,
    verticalScrollbarSize: 10,
    horizontalScrollbarSize: 10
  }
}
</script>

<template>
    <div class="editor-container">
    <div class="monaco-container">
      <VueMonacoEditor
        v-model:value="jsonContent"
        :options="editorOptions"
        @change="handleJsonChange"
        style="height: 100%; width: 100%;"
      />
    </div>
  </div>
</template>

<style scoped>
.editor-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.monaco-container {
  flex: 1;
  min-height: 0;
  overflow: hidden;
  border: 1px solid #e0e0e0;
}

:deep(.monaco-editor) {
  font-family: 'Courier New', monospace !important;
}

:deep(.monaco-editor .margin) {
  background-color: #f8f9fa !important;
}

:deep(.monaco-editor .monaco-editor-background) {
  background-color: #ffffff !important;
}
</style>