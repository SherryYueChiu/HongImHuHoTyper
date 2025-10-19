<script setup lang="ts">
import { ref } from 'vue'
import VirtualKeyboard from '@/components/VirtualKeyboard.vue'
import ResultDisplay from '@/components/ResultDisplay.vue'
import InputBar from '@/components/InputBar.vue'

// 主要狀態
const inputText = ref('')
const resultText = ref('')
const showResult = ref(false)
const inputBarRef = ref<InstanceType<typeof InputBar>>()
const cursorPosition = ref(0)

// 處理輸入
const handleInput = (text: string) => {
  inputText.value = text
  resultText.value = text
  showResult.value = true
}

// 處理游標位置變化
const handleCursorPosition = (position: number) => {
  cursorPosition.value = position
}

// 處理方音符號輸入
const handlePhoneticInput = (phonetic: string, data?: any) => {
  if (inputBarRef.value) {
    if (phonetic === 'backspace') {
      inputBarRef.value.deleteAtCursor()
    } else if (phonetic === 'space') {
      inputBarRef.value.insertAtCursor(' ')
    } else if (phonetic === 'enter') {
      inputBarRef.value.insertAtCursor('\n')
    } else if (phonetic === 'batch-insert' && data) {
      // 處理批量插入
      inputBarRef.value.insertMultipleAtCursor(data)
    } else {
      inputBarRef.value.insertAtCursor(phonetic)
    }
    
    // 更新結果顯示
    resultText.value = inputText.value
    showResult.value = true
  }
}

// 輸出圖片功能
const exportImage = () => {
  const resultContainer = document.querySelector('.result-container')
  if (!resultContainer) return
  
  // 獲取容器的尺寸和位置
  const containerRect = resultContainer.getBoundingClientRect()
  
  // 創建 canvas
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')!
  
  // 設定畫布大小，添加一些邊距
  const padding = 40
  canvas.width = containerRect.width + padding * 2
  canvas.height = containerRect.height + padding * 2
  
  // 獲取所有字符組元素
  const characterGroups = resultContainer.querySelectorAll('.character-group')
  
  characterGroups.forEach((group) => {
    const groupRect = group.getBoundingClientRect()
    const relativeX = groupRect.left - containerRect.left + padding
    const relativeY = groupRect.top - containerRect.top + padding
    
    // 繪製漢字
    const chineseChar = group.querySelector('.chinese-char')
    if (chineseChar) {
      const charRect = chineseChar.getBoundingClientRect()
      const charX = charRect.left - containerRect.left + padding
      const charY = charRect.top - containerRect.top + padding
      
      ctx.font = '32px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(chineseChar.textContent || '', charX, charY + 32)
    }
    
    // 繪製注音符號
    const phoneticSymbols = group.querySelectorAll('.phonetic-symbol')
    phoneticSymbols.forEach((symbol, index) => {
      const symbolRect = symbol.getBoundingClientRect()
      const symbolX = symbolRect.left - containerRect.left + padding
      const symbolY = symbolRect.top - containerRect.top + padding
      
      ctx.font = '18px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 增加注音符號之間的間距
      const spacing = index * 2
      ctx.fillText(symbol.textContent || '', symbolX, symbolY + 18 + spacing)
    })
    
    // 繪製音調標註
    const toneMarks = group.querySelectorAll('.tone-mark')
    toneMarks.forEach((tone) => {
      if (tone.classList.contains('transparent')) return
      
      const toneRect = tone.getBoundingClientRect()
      const toneX = toneRect.left - containerRect.left + padding
      const toneY = toneRect.top - containerRect.top + padding
      
      // 調整音調位置，進一步降低高度
      ctx.font = '12px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(tone.textContent || '', toneX, toneY + 20)
    })
    
    // 繪製 plain text
    const plainTextChar = group.querySelector('.plain-text-char')
    if (plainTextChar) {
      const textRect = plainTextChar.getBoundingClientRect()
      const textX = textRect.left - containerRect.left + padding
      const textY = textRect.top - containerRect.top + padding
      
      // 調整 plain text 位置，提高高度
      ctx.font = '16px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(plainTextChar.textContent || '', textX, textY + 20)
    }
  })
  
  // 下載圖片
  const link = document.createElement('a')
  link.download = '台語方音符號.png'
  link.href = canvas.toDataURL('image/png')
  link.click()
}
</script>

<template>
  <div class="app">
    <!-- 標題 -->
    <header class="header">
      <h1>台語方音符號輸入法</h1>
      <button v-if="inputText || showResult" @click="exportImage" class="export-btn">
        輸出圖片
      </button>
    </header>

    <!-- 上半部：結果展示區 -->
    <main class="main-content">
      <ResultDisplay 
        :text="resultText" 
        :show="showResult"
      />
    </main>

    <!-- 輸入欄 -->
    <InputBar 
      ref="inputBarRef"
      v-model="inputText"
      @input="handleInput"
      @cursor-position="handleCursorPosition"
    />

    <!-- 下半部：虛擬鍵盤 -->
    <VirtualKeyboard 
      @phonetic-input="handlePhoneticInput"
    />
  </div>
</template>

<style>
/* 全局樣式重置 */
body {
  margin: 0;
  padding: 0;
}

* {
  box-sizing: border-box;
}
</style>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Microsoft JhengHei', 'PingFang TC', 'Helvetica Neue', Arial, sans-serif;
}

.header {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.header h1 {
  color: white;
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
}

.export-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s ease;
  background: #4CAF50;
  color: white;
}

.export-btn:hover {
  background: #45a049;
  transform: translateY(-1px);
}

.main-content {
  flex: 1;
  padding: 1rem;
  overflow-y: auto;
  min-height: 0;
}

@media (max-width: 768px) {
  .header {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .header h1 {
    font-size: 1.2rem;
  }
  
  .export-btn {
    font-size: 0.8rem;
    padding: 0.4rem 0.8rem;
  }
}
</style>
