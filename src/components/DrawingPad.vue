<script setup lang="ts">
import { ref } from 'vue';
import type { Stroke } from '../types/drawing-pad';
import DrawingCanvas from './DrawingCanvas.vue';
import DrawingToolbar from './DrawingToolbar.vue';

const strokes = ref<Stroke[]>([]);
const history = ref<Stroke[][]>([]);
const redoStack = ref<Stroke[][]>([]);

const isErasing = ref(false);
const eraseSize = ref(10);

const handleStrokeAdded = (stroke: Stroke) => {
  if (!stroke.points.length) return;
  history.value.push([...strokes.value]);
  redoStack.value = [];
  strokes.value.push(stroke);
};

function handleUndo() {
  if (history.value.length === 0) return;
  redoStack.value.push([...strokes.value]);
  strokes.value = history.value.pop() || [];
}

function handleRedo() {
  if (redoStack.value.length === 0) return;
  history.value.push([...strokes.value]);
  strokes.value = redoStack.value.pop() || [];
}

function handleClear() {
  if (strokes.value.length) {
    history.value.push([...strokes.value]);
    redoStack.value = [];
    strokes.value = [];
  }
}

function handleEraseToggle() {
  isErasing.value = !isErasing.value;
}
</script>
<template>
  <div>
    <DrawingCanvas
      :strokes="strokes"
      :is-erasing="isErasing"
      :erase-size="eraseSize"
      @stroke-added="handleStrokeAdded"
    />
    <DrawingToolbar
      :on-undo="handleUndo"
      :on-redo="handleRedo"
      :on-clear="handleClear"
      :on-erase-toggle="handleEraseToggle"
      :erase-size="eraseSize"
      :is-erasing="isErasing"
      @update:erase-size="eraseSize = $event"
    />
  </div>
</template>
