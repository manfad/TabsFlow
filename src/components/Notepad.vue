<script lang="ts" setup>
import { ref, nextTick } from 'vue'

// Define props and emits for v-model support
const props = defineProps<{
  lines: string[]
}>()

const emit = defineEmits<{
  'update:lines': [lines: string[]]
}>()

// Use local ref that syncs with props
const lines = ref(props.lines)

const addNewLine = async (index: number) => {
  lines.value.splice(index + 1, 0, '')
  updateLinesImmediate()
  await nextTick()
  const nextDiv = document.querySelector(`[data-line="${index + 1}"]`) as HTMLElement
  if (nextDiv) {
    nextDiv.focus()
  }
}

const handleKeyDown = async (event: KeyboardEvent, index: number) => {
  if (event.key === 'Enter') {
    event.preventDefault()
    await addNewLine(index)
  } else if (event.key === 'Tab') {
    event.preventDefault()
    
    if (event.shiftKey) {
      // Shift+Tab: decrease indentation
      const target = event.target as HTMLElement
      const text = target.textContent || ''
      
      // Check if line starts with tab or spaces
      if (text.startsWith('\t')) {
        // Remove one tab
        const newText = text.substring(1)
        target.textContent = newText
        lines.value[index] = newText
        updateLinesImmediate()
        
        // Set cursor at beginning
        const selection = window.getSelection()
        const range = document.createRange()
        range.setStart(target.firstChild || target, 0)
        range.setEnd(target.firstChild || target, 0)
        selection?.removeAllRanges()
        selection?.addRange(range)
      } else if (text.match(/^[ ]{1,4}/)) {
        // Remove 1-4 spaces from beginning
        const spaces = text.match(/^[ ]*/)?.[0] || ''
        const spacesToRemove = Math.min(spaces.length, 4)
        const newText = text.substring(spacesToRemove)
        target.textContent = newText
        lines.value[index] = newText
        updateLinesImmediate()
        
        // Set cursor at beginning
        const selection = window.getSelection()
        const range = document.createRange()
        range.setStart(target.firstChild || target, 0)
        range.setEnd(target.firstChild || target, 0)
        selection?.removeAllRanges()
        selection?.addRange(range)
      }
    } else {
      // Regular Tab: add indentation
      const selection = window.getSelection()
      if (selection && selection.rangeCount > 0) {
        const range = selection.getRangeAt(0)
        const tabNode = document.createTextNode('\t')
        range.deleteContents()
        range.insertNode(tabNode)
        range.setStartAfter(tabNode)
        range.setEndAfter(tabNode)
        selection.removeAllRanges()
        selection.addRange(range)
      }
    }
  } else if (event.key === 'Backspace') {
    const target = event.target as HTMLElement
    if (target.textContent === '' && lines.value.length > 1) {
      event.preventDefault()
      const prevDiv = document.querySelector(`[data-line="${index - 1}"]`) as HTMLElement
      if (prevDiv && index > 0) {
        lines.value.splice(index, 1)
        updateLinesImmediate()
        await nextTick()
        prevDiv.focus()
        // Move cursor to end of previous line
        const range = document.createRange()
        const selection = window.getSelection()
        range.selectNodeContents(prevDiv)
        range.collapse(false)
        selection?.removeAllRanges()
        selection?.addRange(range)
      }
    }
  }
}

// Debounce function to limit reactive updates
let updateTimeout: any = null

const updateLineContent = (index: number, event: Event) => {
  const target = event.target as HTMLElement
  lines.value[index] = target.textContent || ''
  
  // Debounce the emit to reduce reactive updates
  clearTimeout(updateTimeout)
  updateTimeout = setTimeout(() => {
    emit('update:lines', [...lines.value])
  }, 100)
}

// Update lines immediately for structural changes (used by keyboard handlers)
const updateLinesImmediate = () => {
  clearTimeout(updateTimeout)
  emit('update:lines', [...lines.value])
}
</script>

<template>
  <div class="note-editor">
    <div
      v-for="(_, index) in lines"
      :key="index"
      class="line-container"
    >
      <div class="line-number">{{ index + 1 }}</div>
      <div
        :data-line="index"
        class="note-line"
        contenteditable="true"
        @keydown="handleKeyDown($event, index)"
        @input="updateLineContent(index, $event)"
        spellcheck="false"
      ></div>
    </div>
  </div>
</template>

<style scoped>
.note-editor {
  font-family: 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.5;
  padding: 16px;
  background-color: #f8f9fa;
  border-radius: 8px;
  min-height: 200px;
  max-height: 100%;
  overflow-y: auto;
}

.line-container {
  display: flex;
  align-items: flex-start;
}

.line-container:hover .note-line {
  background-color: rgba(0, 0, 0, 0.02);
}

.line-container:has(.note-line:focus) .note-line {
  background-color: rgba(0, 123, 255, 0.05);
}

.line-number {
  color: #6c757d;
  text-align: right;
  padding-right: 12px;
  min-width: 40px;
  user-select: none;
  flex-shrink: 0;
  padding-top: 2px;
  font-size: 12px;
}

.note-line {
  outline: none;
  padding: 2px 0;
  min-height: 1.5em;
  white-space: pre;
  border: none;
  background: transparent;
  flex: 1;
  min-width: 0;
}

.note-line:empty::before {
  content: '\00a0';
  color: transparent;
}
</style>