<script setup>
import { ref } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    required: true
  },
  colors: {
    type: Array,
    default: () => [
      '#000000', '#ffffff', '#ef4444', '#f97316', '#f59e0b', 
      '#84cc16', '#22c55e', '#10b981', '#06b6d4', '#0ea5e9', 
      '#3b82f6', '#6366f1', '#8b5cf6', '#d946ef', '#f43f5e'
    ]
  }
})

const emit = defineEmits(['update:modelValue', 'add-color'])

const newColor = ref('#ffffff')

const selectColor = (color) => {
  emit('update:modelValue', color)
}

const addColor = () => {
  emit('add-color', newColor.value)
}
</script>

<template>
  <div class="card palette-panel">
    <h3>Palette</h3>
    <div class="colors-grid">
      <button 
        v-for="color in colors" 
        :key="color"
        class="color-swatch"
        :class="{ active: modelValue === color }"
        :style="{ backgroundColor: color }"
        @click="selectColor(color)"
        :aria-label="`Select color ${color}`"
      ></button>
      
      <div class="add-color-wrapper">
        <input 
          type="color" 
          v-model="newColor" 
          class="color-input"
          @change="addColor"
          title="Add custom color"
        />
        <div class="plus-icon">+</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.palette-panel {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

.colors-grid {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-sm);
}

.color-swatch {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 2px solid transparent;
  cursor: pointer;
  transition: transform 0.1s, border-color 0.2s;
  box-shadow: var(--shadow-sm);
}

.color-swatch:hover {
  transform: scale(1.1);
}

.color-swatch.active {
  border-color: var(--text-primary);
  transform: scale(1.1);
  box-shadow: 0 0 0 2px var(--bg-secondary), 0 0 0 4px var(--primary-color);
}

.add-color-wrapper {
  position: relative;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background-color: var(--bg-tertiary);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  overflow: hidden;
  transition: background-color 0.2s;
}

.add-color-wrapper:hover {
  background-color: var(--bg-secondary);
}

.color-input {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  cursor: pointer;
}

.plus-icon {
  font-size: 1.2rem;
  color: var(--text-secondary);
  pointer-events: none;
}
</style>
