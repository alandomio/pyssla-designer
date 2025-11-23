<script setup>
import { ref, reactive, watch } from 'vue'
import ConfigPanel from './components/ConfigPanel.vue'
import ColorPalette from './components/ColorPalette.vue'
import PysslaGrid from './components/PysslaGrid.vue'

// State
const width = ref(20)
const height = ref(20)
const selectedColor = ref('#000000')
const palette = ref([
  '#000000', '#ffffff', '#ef4444', '#f97316', '#f59e0b', 
  '#84cc16', '#22c55e', '#10b981', '#06b6d4', '#0ea5e9', 
  '#3b82f6', '#6366f1', '#8b5cf6', '#d946ef', '#f43f5e'
])

// Grid Data - 2D Array
// Initialize with empty strings (transparent/default)
const createGrid = (w, h) => Array(h).fill().map(() => Array(w).fill(''))
const gridData = ref(createGrid(width.value, height.value))

// History
const history = ref([])
const historyIndex = ref(-1)

const saveState = () => {
  // Remove any future history if we are in the middle of the stack
  if (historyIndex.value < history.value.length - 1) {
    history.value = history.value.slice(0, historyIndex.value + 1)
  }
  
  // Deep copy grid
  const snapshot = JSON.parse(JSON.stringify(gridData.value))
  history.value.push(snapshot)
  historyIndex.value++
  
  // Limit history size (optional, e.g., 50 steps)
  if (history.value.length > 50) {
    history.value.shift()
    historyIndex.value--
  }
}

const undo = () => {
  if (historyIndex.value > 0) {
    historyIndex.value--
    gridData.value = JSON.parse(JSON.stringify(history.value[historyIndex.value]))
  }
}

const redo = () => {
  if (historyIndex.value < history.value.length - 1) {
    historyIndex.value++
    gridData.value = JSON.parse(JSON.stringify(history.value[historyIndex.value]))
  }
}

// Initialize history
saveState()

// Handlers
const handleResize = () => {
  saveState() // Save BEFORE resizing? Or after? Usually user wants to undo the resize.
              // Let's save the state *before* the change happens.
              // Actually, watch triggers after change. We need to capture before.
              // But resizing is destructive. Let's save the *result* of the resize as a new state.
              // Wait, if we save result, undo will go back to previous size. Correct.
              
  const newGrid = createGrid(width.value, height.value)
  
  // Preserve existing data where possible
  for (let y = 0; y < Math.min(height.value, gridData.value.length); y++) {
    for (let x = 0; x < Math.min(width.value, gridData.value[0].length); x++) {
      newGrid[y][x] = gridData.value[y][x]
    }
  }
  
  gridData.value = newGrid
  saveState()
}

// Watch for size changes to resize grid
// Note: This watch is tricky with history. 
// If we use the watch, every keystroke in input might trigger resize.
// But ConfigPanel uses v-model.number with @change (lazy), so it updates on blur/enter.
// So it's fine.
watch([width, height], handleResize)

const handleStrokeStart = () => {
  saveState()
}

const handlePaint = ({ x, y, color }) => {
  gridData.value[y][x] = color
}

const handleClear = () => {
  if (confirm('Are you sure you want to clear the grid?')) {
    saveState()
    gridData.value = createGrid(width.value, height.value)
    saveState() // Save the cleared state
  }
}

const handleAddColor = (newColor) => {
  if (!palette.value.includes(newColor)) {
    palette.value.push(newColor)
    selectedColor.value = newColor
  }
}

const handleSave = () => {
  const data = {
    version: 1,
    width: width.value,
    height: height.value,
    palette: palette.value,
    grid: gridData.value
  }
  
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `pyssla-pattern-${new Date().toISOString().slice(0,10)}.json`
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

const handleLoad = (data) => {
  if (!data.grid || !data.width || !data.height) {
    alert('Invalid pattern file')
    return
  }
  
  width.value = data.width
  height.value = data.height
  
  if (data.palette) {
    palette.value = data.palette
  }
  
  // Ensure grid data matches dimensions (basic safety)
  // We can just directly assign if we trust the file, or map it to be safe
  // For now, direct assignment but let's make sure it's reactive
  gridData.value = data.grid
  
  // Reset history on load
  history.value = []
  historyIndex.value = -1
  saveState()
}

// Keyboard Shortcuts
import { onMounted, onUnmounted } from 'vue'

const handleKeydown = (e) => {
  if ((e.ctrlKey || e.metaKey) && e.key === 'z') {
    e.preventDefault()
    if (e.shiftKey) {
      redo()
    } else {
      undo()
    }
  } else if ((e.ctrlKey || e.metaKey) && e.key === 'y') {
    e.preventDefault()
    redo()
  }
}

onMounted(() => window.addEventListener('keydown', handleKeydown))
onUnmounted(() => window.removeEventListener('keydown', handleKeydown))
</script>

<template>
  <div class="container">
    <header class="header">
      <h1>Pyssla Creator</h1>
      <p class="subtitle">Design your bead patterns with ease</p>
    </header>
    
    <div class="main-layout">
      <aside class="sidebar">
        <ConfigPanel 
          v-model:width="width" 
          v-model:height="height" 
          :can-undo="historyIndex > 0"
          :can-redo="historyIndex < history.length - 1"
          @clear="handleClear"
          @save="handleSave"
          @load="handleLoad"
          @undo="undo"
          @redo="redo"
        />
        
        <ColorPalette 
          v-model="selectedColor" 
          :colors="palette"
          @add-color="handleAddColor"
        />
        
        <div class="card instructions">
          <h3>Instructions</h3>
          <ul>
            <li>Select a color from the palette.</li>
            <li>Click or drag on the grid to paint.</li>
            <li>Use the config panel to resize.</li>
            <li>Add custom colors with the + button.</li>
          </ul>
        </div>
      </aside>
      
      <main class="workspace">
        <div class="grid-wrapper">
          <PysslaGrid 
            :grid-data="gridData"
            :width="width"
            :height="height"
            :selected-color="selectedColor"
            @paint="handlePaint"
            @stroke-start="handleStrokeStart"
          />
        </div>
      </main>
    </div>
  </div>
</template>

<style scoped>
.header {
  text-align: center;
  margin-bottom: var(--spacing-xl);
}

.subtitle {
  color: var(--text-secondary);
  font-size: 1.1rem;
  margin-top: -var(--spacing-md);
}

.main-layout {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: var(--spacing-xl);
  align-items: start;
}

.sidebar {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-lg);
  position: sticky;
  top: var(--spacing-md);
}

.workspace {
  background-color: var(--bg-secondary);
  border-radius: var(--radius-lg);
  padding: var(--spacing-xl);
  border: 1px solid var(--border-color);
  min-height: 600px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  overflow: auto;
}

.grid-wrapper {
  /* Ensure grid is centered if smaller than workspace */
  margin: auto;
}

.instructions ul {
  padding-left: var(--spacing-lg);
  color: var(--text-secondary);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
}

@media (max-width: 768px) {
  .main-layout {
    grid-template-columns: 1fr;
  }
  
  .sidebar {
    position: static;
  }
}
</style>
