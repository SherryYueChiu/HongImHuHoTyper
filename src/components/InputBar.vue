<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'

interface Props {
  modelValue: string
}

interface Emits {
  (e: 'update:modelValue', value: string): void
  (e: 'input', value: string): void
  (e: 'cursor-position', position: number): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

const inputRef = ref<HTMLInputElement>()
const isFocused = ref(false)
const cursorPosition = ref(0)

// 監聽輸入變化
watch(() => props.modelValue, (newValue) => {
  if (inputRef.value) {
    inputRef.value.value = newValue
  }
})

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  const value = target.value
  cursorPosition.value = target.selectionStart || 0
  emit('update:modelValue', value)
  emit('input', value)
  emit('cursor-position', cursorPosition.value)
}

const handleFocus = () => {
  isFocused.value = true
}

const handleBlur = () => {
  isFocused.value = false
}

const handleClick = (event: MouseEvent) => {
  const target = event.target as HTMLInputElement
  cursorPosition.value = target.selectionStart || 0
  emit('cursor-position', cursorPosition.value)
}

const handleKeyUp = (event: KeyboardEvent) => {
  const target = event.target as HTMLInputElement
  cursorPosition.value = target.selectionStart || 0
  emit('cursor-position', cursorPosition.value)
}

const clearInput = () => {
  emit('update:modelValue', '')
  emit('input', '')
  cursorPosition.value = 0
  emit('cursor-position', 0)
  if (inputRef.value) {
    inputRef.value.focus()
  }
}

// 暴露方法給父組件使用
const insertAtCursor = (text: string) => {
  if (inputRef.value) {
    const currentValue = inputRef.value.value
    const position = cursorPosition.value
    const newValue = currentValue.slice(0, position) + text + currentValue.slice(position)
    
    // 更新游標位置
    cursorPosition.value = position + text.length
    
    emit('update:modelValue', newValue)
    emit('input', newValue)
    emit('cursor-position', cursorPosition.value)
    
    // 使用 nextTick 確保 DOM 更新後再設置游標位置
    nextTick(() => {
      if (inputRef.value) {
        inputRef.value.setSelectionRange(cursorPosition.value, cursorPosition.value)
      }
    })
  }
}

// 批量插入字符的方法
const insertMultipleAtCursor = (texts: string[]) => {
  if (inputRef.value && texts.length > 0) {
    const currentValue = inputRef.value.value
    const position = cursorPosition.value
    const combinedText = texts.join('')
    const newValue = currentValue.slice(0, position) + combinedText + currentValue.slice(position)
    
    // 更新游標位置
    cursorPosition.value = position + combinedText.length
    
    emit('update:modelValue', newValue)
    emit('input', newValue)
    emit('cursor-position', cursorPosition.value)
    
    // 使用 nextTick 確保 DOM 更新後再設置游標位置
    nextTick(() => {
      if (inputRef.value) {
        inputRef.value.setSelectionRange(cursorPosition.value, cursorPosition.value)
      }
    })
  }
}

const deleteAtCursor = () => {
  if (inputRef.value && cursorPosition.value > 0) {
    const currentValue = inputRef.value.value
    const position = cursorPosition.value
    const newValue = currentValue.slice(0, position - 1) + currentValue.slice(position)
    
    // 更新游標位置
    cursorPosition.value = position - 1
    
    emit('update:modelValue', newValue)
    emit('input', newValue)
    emit('cursor-position', cursorPosition.value)
    
    // 使用 nextTick 確保 DOM 更新後再設置游標位置
    nextTick(() => {
      if (inputRef.value) {
        inputRef.value.setSelectionRange(cursorPosition.value, cursorPosition.value)
      }
    })
  }
}

// 暴露方法
defineExpose({
  insertAtCursor,
  insertMultipleAtCursor,
  deleteAtCursor,
  cursorPosition
})
</script>

<template>
  <div class="input-bar" :class="{ focused: isFocused }">
    <div class="input-container">
      <div class="input-wrapper">
        <textarea
          ref="inputRef"
          :value="modelValue"
          @input="handleInput"
          @focus="handleFocus"
          @blur="handleBlur"
          @click="handleClick"
          @keyup="handleKeyUp"
          placeholder="請輸入漢字或英文..."
          class="text-input"
          rows="3"
        ></textarea>
        <button 
          v-if="modelValue" 
          @click="clearInput" 
          class="clear-button"
          type="button"
        >
          ✕
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.input-bar {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1rem;
  transition: all 0.3s ease;
}

.input-bar.focused {
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.input-container {
  max-width: 600px;
  margin: 0 auto;
}

.input-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  color: #333;
  font-weight: 500;
  font-size: 0.9rem;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.text-input {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  font-family: 'Microsoft JhengHei', 'PingFang TC', sans-serif;
  background: white;
  transition: all 0.2s ease;
  outline: none;
  resize: vertical;
  min-height: 3rem;
  max-height: 8rem;
}

.text-input:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.text-input::placeholder {
  color: #999;
}

.clear-button {
  position: absolute;
  right: 0.5rem;
  background: #f5f5f5;
  border: none;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 0.8rem;
  color: #666;
  transition: all 0.2s ease;
}

.clear-button:hover {
  background: #e0e0e0;
  color: #333;
  transform: scale(1.1);
}

@media (max-width: 768px) {
  .input-bar {
    padding: 0.75rem;
  }
  
  .text-input {
    font-size: 16px; /* 防止 iOS 縮放 */
    padding: 0.875rem 1rem;
  }
  
  .input-label {
    font-size: 0.85rem;
  }
}
</style>
