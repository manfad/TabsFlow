<script lang="ts" setup>
import { computed } from 'vue'
import { VueMonacoEditor } from '@guolao/vue-monaco-editor'

// Define props and emits for v-model support
const props = defineProps<{
  lines: string[]
}>()

const emit = defineEmits<{
  'update:lines': [lines: string[]]
}>()

// Convert lines array to/from Monaco's string format
const content = computed({
  get: () => props.lines.join('\n'),
  set: (value: string) => {
    const newLines = value.split('\n')
    emit('update:lines', newLines)
  }
})

// Monaco editor options
const editorOptions = {
  theme: 'vs',
  language: 'plaintext',
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

// Handle content changes
const handleChange = (value: string) => {
  content.value = value
}


</script>

<template>
  <div class="editor-container">
    <div class="monaco-container">
      <VueMonacoEditor
        v-model:value="content"
        :options="editorOptions"
        @change="handleChange"
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