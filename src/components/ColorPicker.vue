<script setup lang="ts">
import { ref, watch } from 'vue'

interface Props {
  color: string
  alpha?: number
  label?: string
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:color', value: string): void
  (e: 'update:alpha', value: number): void
}>()

const internalColor = ref(props.color)
const internalAlpha = ref(props.alpha ?? 1)

watch(() => props.color, (v) => {
  if (v !== internalColor.value) internalColor.value = v
})

watch(() => props.alpha, (v) => {
  if (typeof v === 'number' && v !== internalAlpha.value) internalAlpha.value = v
})

const onColorInput = (e: Event) => {
  const value = (e.target as HTMLInputElement).value
  internalColor.value = value
  emit('update:color', value)
}

const onAlphaInput = (e: Event) => {
  const value = Number((e.target as HTMLInputElement).value)
  internalAlpha.value = value
  emit('update:alpha', value)
}

const presetColors = ['#FFFFFF','#000000','#8B7D6B','#A8A8A8','#D4B5A0','#C7B299','#B8A082','#9B8B7A','#E8D5C4','#F5F1EB']

const applyPreset = (value: string) => {
  internalColor.value = value
  emit('update:color', value)
}
</script>

<template>
  <div class="color-picker">
    <label v-if="label" class="cp-label">{{ label }}</label>
    <div class="cp-row">
      <input type="color" :value="internalColor" @input="onColorInput" class="cp-color" />
      <input type="text" :value="internalColor" @input="onColorInput" class="cp-text" />
    </div>
    <div class="cp-row">
      <input type="range" min="0" max="1" step="0.01" :value="internalAlpha" @input="onAlphaInput" class="cp-alpha" />
      <span class="cp-alpha-val">{{ Math.round((internalAlpha ?? 1) * 100) }}%</span>
    </div>
    <div class="cp-presets">
      <button v-for="c in presetColors" :key="c" class="cp-preset" :style="{ backgroundColor: c }" @click.prevent="applyPreset(c)" />
    </div>
  </div>
  
</template>

<style scoped>
.color-picker {
  display: flex;
  flex-direction: column;
  gap: .5rem;
}

.cp-label {
  font-weight: bold;
  color: #333;
}

.cp-row {
  display: flex;
  align-items: center;
  gap: .5rem;
}

.cp-color {
  width: 40px;
  height: 32px;
  padding: 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: #fff;
}

.cp-text {
  flex: 1;
  height: 32px;
  padding: 0 .5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.cp-alpha {
  flex: 1;
  height: 4px;
  border-radius: 2px;
  background: #ddd;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.cp-alpha::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #667eea;
  cursor: pointer;
}

.cp-alpha::-moz-range-thumb {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #667eea;
  cursor: pointer;
  border: none;
}

.cp-alpha-val {
  min-width: 40px;
  text-align: right;
  font-size: 0.9rem;
  color: #666;
}

.cp-presets {
  display: grid;
  grid-template-columns: repeat(10, 20px);
  gap: 6px;
}

.cp-preset {
  width: 20px;
  height: 20px;
  border-radius: 4px;
  border: 1px solid #ddd;
  cursor: pointer;
}
</style>

