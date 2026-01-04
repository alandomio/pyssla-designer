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
const currentDrawingColor = ref(null)

const startDrawing = (x, y, event) => {
  // 0 = Left click (Paint), 2 = Right click (Erase)
  if (event.button !== 0 && event.button !== 2) return
  
  emit('stroke-start')
  isDrawing.value = true
  
  // Set the color for this stroke
  currentDrawingColor.value = event.button === 2 ? '' : props.selectedColor
  
  paint(x, y, currentDrawingColor.value)
  window.addEventListener('mouseup', stopDrawing)
}

const stopDrawing = () => {
  isDrawing.value = false
  currentDrawingColor.value = null
  window.removeEventListener('mouseup', stopDrawing)
}

const handleMouseEnter = (x, y) => {
  if (isDrawing.value) {
    paint(x, y, currentDrawingColor.value)
  }
}

const paint = (x, y, color) => {
  // Avoid unnecessary emits if color is same
  if (props.gridData[y][x] !== color) {
    emit('paint', { x, y, color })
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
    @contextmenu.prevent
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
        @mousedown.prevent="startDrawing(x, y, $event)"
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
