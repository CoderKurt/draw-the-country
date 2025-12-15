<script setup lang="ts">
const props = defineProps<{
  onUndo: () => void;
  onRedo: () => void;
  onClear: () => void;
  onEraseToggle: () => void;
  eraseSize: number;
  isErasing: boolean;
}>();

const emit = defineEmits<{
  (e: 'update:erase-size', value: number): void;
}>();
</script>
<template>
  <div class="toolbar">
    <button @click="props.onUndo">Undo</button>
    <button @click="props.onRedo">Redo</button>
    <button @click="props.onClear">Clear</button>
    <button @click="props.onEraseToggle">Erase</button>
    <div v-if="props.isErasing" class="size-control">
      <label>
        Erase Size:
        <input
          type="range"
          :min="1"
          :max="50"
          :value="props.eraseSize"
          @input="
            emit(
              'update:erase-size',
              Number(($event.target as HTMLInputElement).value)
            )
          "
        />
        <span>{{ props.eraseSize }}</span>
      </label>
    </div>
  </div>
</template>

<style scoped>
.toolbar {
  display: flex;
  gap: 10px;
  align-items: center;
  padding: 10px;
}

.size-control label {
  display: flex;
  align-items: center;
  gap: 8px;
}

.size-control input[type='range'] {
  width: 100px;
}
</style>
