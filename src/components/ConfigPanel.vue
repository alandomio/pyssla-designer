<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  width: {
    type: Number,
    required: true
  },
  height: {
    type: Number,
    required: true
  },
  canUndo: Boolean,
  canRedo: Boolean
})

const emit = defineEmits(['update:width', 'update:height', 'clear', 'save', 'load', 'undo', 'redo'])

const localWidth = ref(props.width)
const localHeight = ref(props.height)
const fileInput = ref(null)

// Sync local state with props if they change externally
watch(() => props.width, (newVal) => localWidth.value = newVal)
watch(() => props.height, (newVal) => localHeight.value = newVal)

const handleWidthChange = () => {
  if (localWidth.value > 0) {
    emit('update:width', localWidth.value)
  }
}

const handleHeightChange = () => {
  if (localHeight.value > 0) {
    emit('update:height', localHeight.value)
  }
}

const triggerLoad = () => {
  fileInput.value.click()
}

const handleFileLoad = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      emit('load', data)
    } catch (err) {
      alert('Failed to load file: Invalid JSON')
    }
    // Reset input so same file can be loaded again if needed
    event.target.value = ''
  }
  reader.readAsText(file)
}
</script>

<template>
  <div class="card config-panel">
    <h3>Configuration</h3>
    <div class="controls">
      <div class="control-group">
        <label for="width">Width</label>
        <input 
          id="width" 
          type="number" 
          v-model.number="localWidth" 
          @change="handleWidthChange"
          class="input"
          min="1"
          max="100"
        />
      </div>
      <div class="control-group">
        <label for="height">Height</label>
        <input 
          id="height" 
          type="number" 
          v-model.number="localHeight" 
          @change="handleHeightChange"
          class="input"
          min="1"
          max="100"
        />
      </div>
      <div class="actions">
        <div class="history-controls">
          <button class="btn" :disabled="!canUndo" @click="$emit('undo')" title="Undo (Ctrl+Z)">
            ↩ Undo
          </button>
          <button class="btn" :disabled="!canRedo" @click="$emit('redo')" title="Redo (Ctrl+Y)">
            Redo ↪
          </button>
        </div>
        <button class="btn btn-primary" @click="$emit('save')">Save JSON</button>
        <button class="btn" @click="triggerLoad">Load JSON</button>
        <button class="btn btn-danger" @click="$emit('clear')">Clear Grid</button>
        <input 
          ref="fileInput"
          type="file" 
          accept=".json"
          style="display: none" 
          @change="handleFileLoad"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.config-panel {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

.controls {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

.actions {
  display: flex;
  gap: var(--spacing-md);
  flex-wrap: wrap;
  margin-top: var(--spacing-sm);
}

.history-controls {
  display: flex;
  gap: var(--spacing-sm);
  width: 100%;
}

.history-controls .btn {
  flex: 1;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
}

.btn-danger {
  background-color: rgba(239, 68, 68, 0.1);
  color: var(--danger-color);
  border: 1px solid var(--danger-color);
}

.btn-danger:hover {
  background-color: var(--danger-color);
  color: white;
}
</style>
