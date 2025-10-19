<script setup lang="ts">
import { ref, computed } from 'vue'

interface Emits {
  (e: 'phonetic-input', phonetic: string, data?: any): void
}

const emit = defineEmits<Emits>()

// 鍵盤模式狀態
const keyboardMode = ref<'taiwanese' | 'mandarin'>('taiwanese')

// 華語注音符號鍵盤佈局（五行版本）
const mandarinLayout = [
  ['ㄅ', 'ㄉ', 'ˇ', 'ˋ', 'ㄓ', 'ˊ', '˙', 'ㄚ', 'ㄞ', 'ㄢ', 'ㄦ'],
  ['ㄆ', 'ㄊ', 'ㄍ', 'ㄐ', 'ㄔ', 'ㄗ', 'ㄧ', 'ㄛ', 'ㄟ', 'ㄣ'],
  ['ㄇ', 'ㄋ', 'ㄎ', 'ㄑ', 'ㄕ', 'ㄘ', 'ㄨ', 'ㄜ', 'ㄠ', 'ㄤ'],
  ['ㄈ', 'ㄌ', 'ㄏ', 'ㄒ', 'ㄖ', 'ㄙ', 'ㄩ', 'ㄝ', 'ㄡ', 'ㄥ'],
  ['switch', 'space', 'backspace', 'enter']
]

// 台語方音符號鍵盤佈局（五行版本）
const taiwaneseLayout = [
  ['ㄅ', 'ㄉ', '˪', 'ˋ', 'ˊ', '˙', 'ㄚ', 'ㄞ', 'ㄢ', '˫'],
  ['ㄆ', 'ㄊ', 'ㄍ', 'ㄐ', 'ㄗ', 'ㄧ', 'ㆦ', 'ㄛ', 'ㄣ'],
  ['ㄇ', 'ㄋ', 'ㄎ', 'ㄑ', 'ㄘ', 'ㄨ', 'ㄜ', 'ㄠ', 'ㄤ'],
  ['ㄫ', 'ㄌ', 'ㄏ', 'ㄒ', 'ㄙ', 'ɤ', 'ㆤ', 'ㆲ', 'ㄥ'],
  ['switch', 'ㆴ', 'ㆵ', 'ㆻ', 'ㆷ', 'space', 'backspace', 'enter']
]

// 當前鍵盤佈局
const keyboardLayout = computed(() => {
  return keyboardMode.value === 'taiwanese' ? taiwaneseLayout : mandarinLayout
})

// 特殊符號組合映射
const specialSymbolMap: Record<string, string> = {
  // ɤ 組合符號
  'ㄅɤ': 'ㆠ',
  'ㄍɤ': 'ㆣ', 
  'ㄐɤ': 'ㆢ',
  'ㄗɤ': 'ㆡ',
  'ㄚɤ': 'ㆩ',
  'ㄛɤ': 'ㆧ',
  'ㆤɤ': 'ㆥ',
  'ㄝɤ': 'ㆥ',
  'ㄞɤ': 'ㆮ',
  'ㄠɤ': 'ㆯ',
  'ㄧɤ': 'ㆪ',
  'ㄨɤ': 'ㆫ',
  'ㄇɤ': 'ㆬ',
  'ㄫɤ': 'ㆭ',
  // 複合符號
  'ㄚㄇ': 'ㆰ',
  'ㄛㄇ': 'ㆱ',
  'ㆦㄇ': 'ㆱ'
}

// 輸入緩衝區，用於處理組合輸入
const inputBuffer = ref('')

const handleKeyClick = (key: string) => {
  // 觸覺反饋（如果支援）
  if (navigator.vibrate) {
    navigator.vibrate(10)
  }
  
  // 處理切換鍵
  if (key === 'switch') {
    keyboardMode.value = keyboardMode.value === 'taiwanese' ? 'mandarin' : 'taiwanese'
    inputBuffer.value = '' // 清除緩衝區
    return
  }
  
  // 處理特殊鍵
  if (key === 'space' || key === 'backspace' || key === 'enter') {
    // 先輸出緩衝區中的字符（如果有）
    if (inputBuffer.value) {
      // 使用批量插入來避免游標位置問題
      const charsToOutput = []
      for (let i = 0; i < inputBuffer.value.length; i++) {
        const char = inputBuffer.value[i]
        if (char) {
          charsToOutput.push(char)
        }
      }
      // 將特殊鍵轉換為實際字符
      if (key === 'space') {
        charsToOutput.push(' ')
      } else if (key === 'enter') {
        charsToOutput.push('\n')
      } else if (key === 'backspace') {
        // 退格鍵需要特殊處理，不能加入批量插入
        // 先輸出緩衝區中的字符
        for (let i = 0; i < inputBuffer.value.length; i++) {
          const char = inputBuffer.value[i]
          if (char) {
            emit('phonetic-input', char)
          }
        }
        // 然後執行退格操作
        emit('phonetic-input', 'backspace')
        inputBuffer.value = ''
        return
      } else {
        charsToOutput.push(key)
      }
      
      emit('phonetic-input', 'batch-insert', charsToOutput)
      inputBuffer.value = '' // 清除緩衝區
    } else {
      // 沒有緩衝區，直接處理特殊鍵
      emit('phonetic-input', key)
    }
    return
  }
  
  // 簡化的組合處理
  const newBuffer = inputBuffer.value + key
  console.log('Processing key:', key, 'buffer:', inputBuffer.value, 'newBuffer:', newBuffer)
  
  // 檢查是否為完整的組合
  if (specialSymbolMap[newBuffer]) {
    const combinedSymbol = specialSymbolMap[newBuffer]
    console.log('Found combination:', newBuffer, '->', combinedSymbol)
    
    // 先輸出緩衝區中除了最後一個字符外的所有字符
    if (inputBuffer.value.length > 1) {
      const charsToOutput = []
      for (let i = 0; i < inputBuffer.value.length - 1; i++) {
        const char = inputBuffer.value[i]
        if (char) {
          charsToOutput.push(char)
        }
      }
      charsToOutput.push(combinedSymbol)
      console.log('Combination output:', charsToOutput)
      emit('phonetic-input', 'batch-insert', charsToOutput)
    } else {
      // 直接替換，使用批量插入確保順序
      console.log('Direct replacement with:', combinedSymbol)
      emit('phonetic-input', 'batch-insert', [combinedSymbol])
    }
    inputBuffer.value = ''
  } else {
    // 檢查是否為組合的開始
    const possibleCombinations = Object.keys(specialSymbolMap).filter(combo => 
      combo.startsWith(newBuffer) && combo !== newBuffer
    )
    console.log('Possible combinations for', newBuffer, ':', possibleCombinations)
    
    if (possibleCombinations.length > 0) {
      // 可能是組合的開始，加入緩衝區
      console.log('Adding to buffer:', newBuffer)
      inputBuffer.value = newBuffer
    } else {
      // 檢查當前字符是否可能是組合的開始
      const singleCharCombinations = Object.keys(specialSymbolMap).filter(combo => 
        combo.startsWith(key) && combo !== key
      )
      console.log('Single char combinations for', key, ':', singleCharCombinations)
      
      if (singleCharCombinations.length > 0) {
        // 當前字符可能是組合的開始
        console.log('Single char combination start:', key)
        // 先輸出緩衝區中的字符
        if (inputBuffer.value) {
          for (let i = 0; i < inputBuffer.value.length; i++) {
            const char = inputBuffer.value[i]
            if (char) {
              emit('phonetic-input', char)
            }
          }
        }
        // 將新字符加入緩衝區
        inputBuffer.value = key
      } else {
        // 不是組合，輸出緩衝區中的字符和新字符
        console.log('Not a combination, outputting')
        if (inputBuffer.value) {
          const charsToOutput = [...inputBuffer.value, key]
          emit('phonetic-input', 'batch-insert', charsToOutput)
        } else {
          emit('phonetic-input', key)
        }
        inputBuffer.value = ''
      }
    }
  }
}

// 處理長按（用於 backspace）
const handleKeyLongPress = (key: string) => {
  if (key === 'backspace') {
    const interval = setInterval(() => {
      emit('phonetic-input', 'backspace')
    }, 100)
    
    const stopLongPress = () => {
      clearInterval(interval)
      document.removeEventListener('mouseup', stopLongPress)
      document.removeEventListener('touchend', stopLongPress)
    }
    
    document.addEventListener('mouseup', stopLongPress)
    document.addEventListener('touchend', stopLongPress)
  }
}

// 獲取按鍵樣式類別
const getKeyClass = (key: string) => {
  const specialKeys = ['space', 'backspace', 'enter']
  const toneKeys = ['ˊ', 'ˇ', 'ˋ', '˙', 'ˉ', '˪', '˫', 'ㆴ', 'ㆵ', 'ㆻ', 'ㆷ', '.']
  
  if (key === 'switch') {
    return 'switch-key'
  }
  
  if (specialKeys.includes(key)) {
    return 'special-key'
  }
  
  if (toneKeys.includes(key)) {
    return 'tone-key'
  }
  
  if (key === 'ɤ') {
    return 'combination-key'
  }
  
  return 'normal-key'
}

// 獲取按鍵顯示文字
const getKeyDisplay = (key: string) => {
  switch (key) {
    case 'space':
      return '空白'
    case 'backspace':
      return '⌫'
    case 'enter':
      return '↵'
    case 'switch':
      return keyboardMode.value === 'taiwanese' ? '台語' : '華語'
    default:
      return key
  }
}
</script>

<template>
  <div class="virtual-keyboard">
    <div class="keyboard-container">
      <!-- 主要鍵盤 -->
      <div class="keyboard-grid">
        <div
          v-for="(row, rowIndex) in keyboardLayout"
          :key="rowIndex"
          class="keyboard-row"
        >
          <template v-for="(key, keyIndex) in row" :key="`${rowIndex}-${keyIndex}`">
            <button
              v-if="key !== ''"
              @click="handleKeyClick(key)"
              @mousedown="handleKeyLongPress(key)"
              @touchstart="handleKeyLongPress(key)"
              :class="getKeyClass(key)"
              class="keyboard-key"
            >
              {{ getKeyDisplay(key) }}
            </button>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.virtual-keyboard {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1rem;
  min-height: 200px;
  flex-shrink: 0;
}

.keyboard-container {
  max-width: 800px;
  margin: 0 auto;
}

.keyboard-grid {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  align-items: center;
}

.keyboard-row {
  display: flex;
  gap: 0.3rem;
  justify-content: center;
  flex-wrap: wrap;
  margin-bottom: 0.3rem;
}

.keyboard-key {
  min-width: 30px;
  min-height: 30px;
  padding: 0.3rem;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  background: white;
  color: #333;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s ease;
  user-select: none;
  touch-action: manipulation;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Microsoft JhengHei', 'PingFang TC', sans-serif;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.keyboard-key:hover {
  background: #f0f0f0;
  border-color: #ccc;
  transform: translateY(-1px);
}

.keyboard-key:active {
  transform: translateY(0);
  background: #e0e0e0;
}

.keyboard-key.special-key {
  background: #667eea;
  color: white;
  border-color: #5a6fd8;
  font-weight: 600;
  min-width: 40px;
}

.keyboard-key.special-key:hover {
  background: #5a6fd8;
  border-color: #4c5bc4;
}

.keyboard-key.tone-key {
  background: #ff6b6b;
  color: white;
  border-color: #ff5252;
  font-weight: 600;
  min-width: 25px;
}

.keyboard-key.tone-key:hover {
  background: #ff5252;
  border-color: #ff1744;
}

.keyboard-key.combination-key {
  background: #9c27b0;
  color: white;
  border-color: #8e24aa;
  font-weight: 600;
  min-width: 25px;
}

.keyboard-key.combination-key:hover {
  background: #8e24aa;
  border-color: #7b1fa2;
}

.keyboard-key.switch-key {
  background: #4CAF50;
  color: white;
  border-color: #45a049;
  font-weight: 600;
  min-width: 50px;
  font-size: 0.7rem;
}

.keyboard-key.switch-key:hover {
  background: #45a049;
  border-color: #3d8b40;
}


@media (max-width: 768px) {
  .virtual-keyboard {
    padding: 0.75rem;
  }
  
  .keyboard-key {
    min-width: 35px;
    min-height: 35px;
    font-size: 0.9rem;
    padding: 0.4rem;
  }
  
  .keyboard-key.special-key {
    min-width: 50px;
  }
  
  .keyboard-key.tone-key {
    min-width: 30px;
  }
  
  .keyboard-row {
    gap: 0.3rem;
  }
}

@media (max-width: 480px) {
  .keyboard-key {
    min-width: 30px;
    min-height: 30px;
    font-size: 0.8rem;
    padding: 0.3rem;
  }
  
  .keyboard-key.special-key {
    min-width: 45px;
  }
  
  .keyboard-key.tone-key {
    min-width: 25px;
  }
  
  .keyboard-row {
    gap: 0.3rem;
  }
}
</style>