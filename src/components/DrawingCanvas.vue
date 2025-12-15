<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import type { Stroke, Point } from '../types/drawing-pad';

const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;

const props = defineProps<{
  strokes: Stroke[];
  isErasing: boolean;
  eraseSize: number;
}>();

const PEN_SIZE = 2;

const emit = defineEmits<{
  (e: 'stroke-added', stroke: Stroke): void;
}>();

watch(
  () => props.strokes,
  () => {
    drawAll();
  },
  { deep: true }
);

let currentStroke: Stroke | null = null;

onMounted(() => {
  if (canvasRef.value) {
    ctx = canvasRef.value.getContext('2d');
    if (ctx) {
      ctx.lineWidth = 2;
      ctx.lineJoin = 'round';
      ctx.lineCap = 'round';
      ctx.strokeStyle = '#000';
    }
  }
});

function getPointerPos(event: PointerEvent): Point {
  if (!canvasRef.value) return { x: 0, y: 0, t: 0 };
  const rect = canvasRef.value.getBoundingClientRect();
  return {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top,
    t: Date.now(),
  };
}

function startStroke(event: PointerEvent): void {
  const point = getPointerPos(event);
  const color = props.isErasing ? '#fff' : '#000';
  const size = props.isErasing ? props.eraseSize : PEN_SIZE;
  currentStroke = {
    points: [point],
    color,
    size,
  };
}

function draw(event: PointerEvent): void {
  if (!currentStroke || !ctx) return;
  const point = getPointerPos(event);
  const lastPoint = currentStroke.points[currentStroke.points.length - 1];
  const size = props.isErasing ? props.eraseSize : PEN_SIZE;
  ctx.strokeStyle = props.isErasing ? '#fff' : '#000';
  ctx.lineWidth = size;
  currentStroke.color = props.isErasing ? '#fff' : '#000';
  currentStroke.size = size;
  if (lastPoint) {
    ctx.beginPath();
    ctx.moveTo(lastPoint.x, lastPoint.y);
    ctx.lineTo(point.x, point.y);
    ctx.stroke();
  }
  currentStroke.points.push(point);
}

function endStroke(): void {
  if (currentStroke) {
    emit('stroke-added', currentStroke);
    currentStroke = null;
  }
}

function drawAll() {
  if (!ctx || !canvasRef.value) return;
  ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
  for (const stroke of props.strokes) {
    if (stroke.points.length < 2) continue;
    ctx.beginPath();
    for (let i = 1; i < stroke.points.length; i++) {
      const prev = stroke.points[i - 1];
      const curr = stroke.points[i];
      if (prev && curr) {
        ctx.moveTo(prev.x, prev.y);
        ctx.lineTo(curr.x, curr.y);
      }
    }
    ctx.strokeStyle = stroke.color;
    ctx.lineWidth = stroke.size;
    ctx.stroke();
  }
}
</script>
<template>
  <canvas
    ref="canvasRef"
    width="800"
    height="600"
    @pointerdown="startStroke"
    @pointermove="draw"
    @pointerup="endStroke"
    @pointercancel="endStroke"
  />
</template>

<style scoped>
canvas {
  border: 1px solid #ccc;
  background-color: #fff;
  touch-action: none; /* prevents scrolling on mobile while drawing */
}
</style>
