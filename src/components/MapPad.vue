<script lang="ts" setup>
import { computed, ref } from 'vue'
import { VueFlow, useVueFlow, Panel } from '@vue-flow/core'
import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'

// Tooltip state
const tooltip = ref({
  show: false,
  x: 0,
  y: 0,
  title: '',
  description: ''
})

const props = defineProps<{
  lines: string[]
}>()

interface ParsedLine {
  level: number
  text: string
  description: string
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
    
    // Split text and description on first colon
    const trimmedText = text.trim()
    const colonIndex = trimmedText.indexOf(':')
    let finalText = trimmedText
    let description = ''
    
    if (colonIndex !== -1) {
      finalText = trimmedText.substring(0, colonIndex).trim()
      description = trimmedText.substring(colonIndex + 1).trim()
    }
    
    return {
      level,
      text: finalText,
      description,
      originalIndex: index
    }
  }).filter(line => line.text !== '') // Remove empty lines
})

// Convert lines to nodes and edges for vue-flow
const flowData = computed(() => {
  const nodeList: any[] = []
  const edgeList: any[] = []
  const lines = parseLines.value
  
  if (lines.length === 0) return { nodes: nodeList, edges: edgeList }
  
  // Track parent nodes for each level
  const levelParents: { [level: number]: string } = {}
  
  lines.forEach((line, index) => {
    const nodeId = `node-${index}`
    const { level, text, description } = line
    
    // Calculate position
    const x = level * 200 + 50
    const y = index * 80 + 50
    
    // Create node
    nodeList.push({
      id: nodeId,
      type: 'default',
      position: { x, y },
      data: { 
        label: description !== '' ? `${text} ℹ️` : text,
        originalLabel: text,
        level: level,
        description: description,
        hasDescription: description !== ''
      },
      style: {
        background: getNodeColor(level),
        border: `2px solid ${getNodeBorderColor(level)}`,
        borderRadius: '8px',
        padding: '10px',
        fontSize: '14px',
        fontWeight: 'bold',
        color: '#333',
        minWidth: '120px',
        textAlign: 'center'
      },
      class: description !== '' ? 'node-with-description' : '',
      // CHANGED: Set horizontal connection points
      sourcePosition: 'right',
      targetPosition: 'left'
    })
    
    // Track this node as parent for its level
    levelParents[level] = nodeId
    
    // Create edge to parent if not root level
    if (level > 0 && levelParents[level - 1]) {
      edgeList.push({
        id: `edge-${levelParents[level - 1]}-${nodeId}`,
        source: levelParents[level - 1],
        target: nodeId,
        // CHANGED: Remove sourceHandle and targetHandle to let Vue Flow use the position settings
        type: 'smoothstep',
        style: {
          stroke: getEdgeColor(level),
          strokeWidth: 2
        },
        markerEnd: {
          type: 'arrowclosed',
          color: getEdgeColor(level)
        }
      })
    }
  })
  
  return { nodes: nodeList, edges: edgeList }
})

const nodes = computed(() => flowData.value.nodes)
const edges = computed(() => flowData.value.edges)

// Color functions for visual hierarchy
const getNodeColor = (level: number) => {
  const colors = [
    '#e3f2fd', // Light blue
    '#f3e5f5', // Light purple  
    '#e8f5e8', // Light green
    '#fff3e0', // Light orange
    '#fce4ec', // Light pink
    '#f1f8e9'  // Light lime
  ]
  return colors[level % colors.length]
}

const getNodeBorderColor = (level: number) => {
  const colors = [
    '#1976d2', // Blue
    '#7b1fa2', // Purple
    '#388e3c', // Green
    '#f57c00', // Orange
    '#c2185b', // Pink
    '#689f38'  // Lime
  ]
  return colors[level % colors.length]
}

const getEdgeColor = (level: number) => {
  return getNodeBorderColor(level - 1)
}

// Layout options
const layoutType = ref<'tree' | 'compact' | 'radial' | 'linear'>('tree')

// Vue Flow setup
const { fitView, setNodes } = useVueFlow()

// Node click handler for tooltip
const onNodeClick = (event: any) => {
  const node = event.node
  const nodeElement = event.event.target.closest('.vue-flow__node')
  
  if (node.data.hasDescription) {
    const rect = nodeElement.getBoundingClientRect()
    tooltip.value = {
      show: true,
      x: rect.right + 10,
      y: rect.top + rect.height / 2,
      title: node.data.originalLabel,
      description: node.data.description
    }
  }
}

// Hide tooltip when clicking elsewhere
const hideTooltip = () => {
  tooltip.value.show = false
}

// Tree layout algorithm
const applyTreeLayout = () => {
  const lines = parseLines.value
  if (lines.length === 0) return
  
  const nodePositions: { [id: string]: { x: number; y: number } } = {}
  const levelCounts: { [level: number]: number } = {}
  const levelPositions: { [level: number]: number } = {}
  
  // Count nodes per level
  lines.forEach(line => {
    levelCounts[line.level] = (levelCounts[line.level] || 0) + 1
  })
  
  // Calculate positions
  lines.forEach((line, index) => {
    const nodeId = `node-${index}`
    const { level } = line
    
    if (!levelPositions[level]) levelPositions[level] = 0
    
    let x: number, y: number
    
    if (layoutType.value === 'tree') {
      // Tree layout: levels go horizontally, spread vertically
      x = level * 250 + 100
      const levelIndex = levelPositions[level]
      const totalAtLevel = levelCounts[level]
      y = (levelIndex - totalAtLevel / 2) * 120 + 300
    } else if (layoutType.value === 'compact') {
      // Compact layout: tighter spacing
      x = level * 180 + 80
      y = levelPositions[level] * 80 + 50
    } else if (layoutType.value === 'radial') {
      // Radial layout: circular arrangement
      const radius = level * 150 + 100
      const angle = (levelPositions[level] / Math.max(levelCounts[level], 1)) * 2 * Math.PI
      x = Math.cos(angle) * radius + 400
      y = Math.sin(angle) * radius + 300
    } else {
      // Linear layout: original simple layout
      x = level * 200 + 50
      y = index * 80 + 50
    }
    
    nodePositions[nodeId] = { x, y }
    levelPositions[level]++
  })
  
  // Update node positions
  const updatedNodes = nodes.value.map(node => ({
    ...node,
    position: nodePositions[node.id] || node.position
  }))
  
  setNodes(updatedNodes)
  
  setTimeout(() => {
    fitView({ padding: 0.2 })
  }, 100)
}

// Auto-fit view when data changes
const onLoad = () => {
  setTimeout(() => {
    applyTreeLayout()
  }, 100)
}

</script>

<template>
  <div class="map-pad">
    <div class="flow-container">
      <VueFlow 
        :nodes="nodes" 
        :edges="edges"
        @vue-flow:load="onLoad"
        @node-click="onNodeClick"
        @pane-click="hideTooltip"
        :default-viewport="{ zoom: 1, x: 0, y: 0 }"
        :min-zoom="0.2"
        :max-zoom="4"
        fit-view-on-init
      >
        <Panel position="top-right" class="controls">
          <div class="control-group">
            <label>Layout:</label>
            <select v-model="layoutType" @change="applyTreeLayout" class="layout-select">
              <option value="tree">Tree</option>
              <option value="compact">Compact</option>
              <option value="radial">Radial</option>
              <option value="linear">Linear</option>
            </select>
          </div>
          <button @click="applyTreeLayout" class="layout-button">
            Auto Layout
          </button>
          <button @click="fitView({ padding: 0.2 })" class="fit-button">
            Fit View
          </button>
        </Panel>
      </VueFlow>
    </div>
    
    <!-- Tooltip -->
    <div 
      v-if="tooltip.show" 
      class="node-tooltip"
      :style="{
        left: tooltip.x + 'px',
        top: tooltip.y + 'px'
      }"
      @click.stop
    >
      <div class="tooltip-title">{{ tooltip.title }}</div>
      <div class="tooltip-description">{{ tooltip.description }}</div>
      <div class="tooltip-arrow"></div>
    </div>
  </div>
</template>

<style scoped>
.map-pad {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  background-color: #f0f8ff;
  border-radius: 8px;
  overflow: hidden;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  background-color: #e3f2fd;
  border-bottom: 1px solid #bbdefb;
}

.header h3 {
  margin: 0;
  color: #1565c0;
  font-size: 16px;
}

.stats {
  font-size: 12px;
  color: #1976d2;
}

.stats span {
  margin-left: 12px;
  padding: 2px 6px;
  background-color: #ffffff;
  border-radius: 4px;
  border: 1px solid #bbdefb;
}

.flow-container {
  flex: 1;
  position: relative;
  background: linear-gradient(45deg, #f8f9fa 25%, transparent 25%), 
              linear-gradient(-45deg, #f8f9fa 25%, transparent 25%), 
              linear-gradient(45deg, transparent 75%, #f8f9fa 75%), 
              linear-gradient(-45deg, transparent 75%, #f8f9fa 75%);
  background-size: 20px 20px;
  background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
}

.controls {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 6px;
  padding: 12px;
  backdrop-filter: blur(4px);
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-width: 150px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.control-group label {
  font-size: 11px;
  font-weight: bold;
  color: #333;
}

.layout-select {
  background: white;
  border: 1px solid #ddd;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
}

.layout-button,
.fit-button {
  background: #1976d2;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
  transition: background 0.2s;
}

.layout-button {
  background: #2e7d32;
}

.layout-button:hover {
  background: #1b5e20;
}

.fit-button:hover {
  background: #1565c0;
}

/* Vue Flow customizations */
:deep(.vue-flow__node) {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

:deep(.vue-flow__edge) {
  z-index: 1;
}

:deep(.vue-flow__handle) {
  background: transparent;
  border: none;
  width: 8px;
  height: 8px;
}

:deep(.vue-flow__handle-right) {
  right: -4px;
  background: rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 50%;
}

:deep(.vue-flow__handle-left) {
  left: -4px;
  background: rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 50%;
}

/* Tooltip Styles */
.node-tooltip {
  position: fixed;
  z-index: 1000;
  background: white;
  border: 2px solid #2196f3;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 12px;
  max-width: 250px;
  font-size: 14px;
  transform: translateY(-50%);
  pointer-events: auto;
}

.tooltip-title {
  font-weight: bold;
  color: #1565c0;
  margin-bottom: 6px;
  font-size: 15px;
}

.tooltip-description {
  color: #333;
  line-height: 1.4;
}

.tooltip-arrow {
  position: absolute;
  left: -8px;
  top: 50%;
  transform: translateY(-50%);
  width: 0;
  height: 0;
  border-top: 8px solid transparent;
  border-bottom: 8px solid transparent;
  border-right: 8px solid #2196f3;
}

.tooltip-arrow::after {
  content: '';
  position: absolute;
  left: 2px;
  top: -6px;
  width: 0;
  height: 0;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-right: 6px solid white;
}

/* Node cursor styling */
:deep(.node-with-description) {
  cursor: pointer !important;
}

:deep(.vue-flow__node.node-with-description .vue-flow__node-default) {
  cursor: pointer !important;
}
</style>