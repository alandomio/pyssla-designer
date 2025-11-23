<script setup>
import { ref } from 'vue'

const props = defineProps({
  gridData: {
    type: Array,
    required: true
  },
  selectedColor: {
    type: String,
    required: true
  },
  width: {
    type: Number,
    required: true
  },
  height: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['paint', 'stroke-start'])

const isDrawing = ref(false)

const startDrawing = (x, y) => {
  emit('stroke-start')
  isDrawing.value = true
  paint(x, y)
  window.addEventListener('mouseup', stopDrawing)
}

const stopDrawing = () => {
  isDrawing.value = false
  window.removeEventListener('mouseup', stopDrawing)
}

const handleMouseEnter = (x, y) => {
  if (isDrawing.value) {
    paint(x, y)
  }
}

const paint = (x, y) => {
  // Avoid unnecessary emits if color is same
  if (props.gridData[y][x] !== props.selectedColor) {
    emit('paint', { x, y, color: props.selectedColor })
  }
}
</script>

<template>
  <div 
    class="grid-container"
    :style="{
      '--grid-width': width,
      '--grid-height': height
    }"
    @mouseleave="stopDrawing"
  >
    <div 
      v-for="(row, y) in gridData" 
      :key="y" 
      class="grid-row"
    >
      <div 
        v-for="(color, x) in row" 
        :key="`${x}-${y}`"
        class="grid-cell"
        :style="{ backgroundColor: color || 'transparent' }"
        @mousedown.prevent="startDrawing(x, y)"
        @mouseenter="handleMouseEnter(x, y)"
      ></div>
    </div>
  </div>
</template>

<style scoped>
.grid-container {
  display: flex;
  flex-direction: column;
  gap: 0;
  background-color: transparent;
  padding: 0;
  border-top: 1px solid #475569;
  border-left: 1px solid #475569;
  border-radius: 0;
  width: fit-content;
  margin: 0 auto;
  user-select: none;
  box-shadow: var(--shadow-lg);
}

.grid-row {
  display: flex;
  gap: 0;
}

.grid-cell {
  width: 20px;
  height: 20px;
  background-color: var(--bg-secondary); /* Default empty color */
  cursor: crosshair;
  transition: background-color 0.05s;
  border-right: 1px solid #475569;
  border-bottom: 1px solid #475569;
  box-sizing: border-box; /* Ensure border is included in width/height */
}

.grid-cell:hover {
  opacity: 0.8;
  box-shadow: inset 0 0 0 2px rgba(255, 255, 255, 0.2);
}
</style>
