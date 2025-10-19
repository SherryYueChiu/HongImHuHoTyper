<script setup lang="ts">
import { ref, computed } from 'vue'
import VirtualKeyboard from '@/components/VirtualKeyboard.vue'
import ResultDisplay from '@/components/ResultDisplay.vue'
import InputBar from '@/components/InputBar.vue'

// 主要狀態
const inputText = ref('')
const resultText = ref('')
const showResult = ref(false)
const inputBarRef = ref<InstanceType<typeof InputBar>>()
const cursorPosition = ref(0)

// 輸出選項
const showExportModal = ref(false)
const exportOptions = ref({
  color: '#FFFFFF',
  weight: 'normal',
  background: 'transparent'
})

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

// 顯示輸出選項窗口
const showExportOptions = () => {
  showExportModal.value = true
}

// 確認輸出
const confirmExport = () => {
  showExportModal.value = false
  exportImage()
}

// 背景預覽樣式
const backgroundPreviewStyle = computed(() => {
  if (exportOptions.value.background === 'transparent') {
    return {
      backgroundColor: 'transparent',
      backgroundImage: 'url("data:image/svg+xml,%3Csvg xmlns=\'http://www.w3.org/2000/svg\' width=\'20\' height=\'20\' viewBox=\'0 0 20 20\'%3E%3Cg fill=\'%23f0f0f0\'%3E%3Crect x=\'0\' y=\'0\' width=\'10\' height=\'10\'/%3E%3Crect x=\'10\' y=\'10\' width=\'10\' height=\'10\'/%3E%3C/g%3E%3C/svg%3E")'
    }
  }
  return {
    backgroundColor: exportOptions.value.background,
    backgroundImage: 'none'
  }
})

// 輸出圖片功能
const exportImage = () => {
  const resultContainer = document.querySelector('.result-container')
  if (!resultContainer) return
  
  // 獲取容器的尺寸和位置
  const containerRect = resultContainer.getBoundingClientRect()
  
  // 創建 canvas
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')!
  
  // 計算實際內容邊界
  const characterGroups = resultContainer.querySelectorAll('.character-group')
  let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity
  
  characterGroups.forEach((group) => {
    // 跳過換行符號，它們會影響寬度計算
    if (group.classList.contains('line-break')) return
    
    const groupRect = group.getBoundingClientRect()
    const relativeX = groupRect.left - containerRect.left
    const relativeY = groupRect.top - containerRect.top
    
    minX = Math.min(minX, relativeX)
    minY = Math.min(minY, relativeY)
    maxX = Math.max(maxX, relativeX + groupRect.width)
    maxY = Math.max(maxY, relativeY + groupRect.height)
  })
  
  // 設定畫布大小，只包含實際內容
  const padding = 10
  const contentWidth = maxX - minX
  const contentHeight = maxY - minY
  
  console.log('Content dimensions:', {
    contentWidth,
    contentHeight,
    minX,
    minY,
    maxX,
    maxY,
    containerWidth: containerRect.width,
    containerHeight: containerRect.height
  })
  
  canvas.width = contentWidth + padding * 2
  canvas.height = contentHeight + padding * 2
  
  // 設置背景
  if (exportOptions.value.background !== 'transparent') {
    ctx.fillStyle = exportOptions.value.background
    ctx.fillRect(0, 0, canvas.width, canvas.height)
  }
  
  characterGroups.forEach((group) => {
    // 跳過換行符號，它們不需要繪製
    if (group.classList.contains('line-break')) return
    
    const groupRect = group.getBoundingClientRect()
    const relativeX = groupRect.left - containerRect.left - minX + padding
    const relativeY = groupRect.top - containerRect.top - minY + padding
    
    // 繪製漢字
    const chineseChar = group.querySelector('.chinese-char')
    if (chineseChar) {
      const charRect = chineseChar.getBoundingClientRect()
      const charX = charRect.left - containerRect.left - minX + padding
      const charY = charRect.top - containerRect.top - minY + padding
      
      ctx.font = `32px "Microsoft JhengHei", "PingFang TC", sans-serif`
      ctx.fillStyle = exportOptions.value.color
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(chineseChar.textContent || '', charX, charY)
    }
    
    // 繪製注音符號
    const phoneticSymbols = group.querySelectorAll('.phonetic-symbol')
    phoneticSymbols.forEach((symbol, index) => {
      const symbolRect = symbol.getBoundingClientRect()
      const symbolX = symbolRect.left - containerRect.left - minX + padding
      const symbolY = symbolRect.top - containerRect.top - minY + padding
      
      ctx.font = `18px "Microsoft JhengHei", "PingFang TC", sans-serif`
      ctx.fillStyle = exportOptions.value.color
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 增加注音符號之間的間距
      const spacing = index * 2
      ctx.fillText(symbol.textContent || '', symbolX, symbolY + spacing)
    })
    
    // 繪製音調標註
    const toneMarks = group.querySelectorAll('.tone-mark')
    toneMarks.forEach((tone) => {
      if (tone.classList.contains('transparent')) return
      
      const toneRect = tone.getBoundingClientRect()
      const toneX = toneRect.left - containerRect.left - minX + padding + 3
      const toneY = toneRect.top - containerRect.top - minY + padding
      
      // 調整音調位置，進一步降低高度
      ctx.font = `12px "Microsoft JhengHei", "PingFang TC", sans-serif`
      ctx.fillStyle = exportOptions.value.color
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(tone.textContent || '', toneX, toneY)
    })
    
    // 繪製 plain text
    const plainTextChar = group.querySelector('.plain-text-char')
    if (plainTextChar) {
      const textRect = plainTextChar.getBoundingClientRect()
      const textX = textRect.left - containerRect.left - minX + padding
      const textY = textRect.top - containerRect.top - minY + padding
      
      // 調整 plain text 位置，提高高度
      ctx.font = `16px "Microsoft JhengHei", "PingFang TC", sans-serif`
      ctx.fillStyle = exportOptions.value.color
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(plainTextChar.textContent || '', textX, textY)
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
      <button v-if="inputText || showResult" @click="showExportOptions" class="export-btn">
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

    <!-- 輸出選項彈出窗口 -->
    <div v-if="showExportModal" class="modal-overlay" @click="showExportModal = false">
      <div class="modal-content" @click.stop>
        <h3>輸出選項</h3>
        
        <div class="option-group">
          <label for="color-select">顏色：</label>
          <div class="color-preview-container">
            <div class="color-preview" :style="{ backgroundColor: exportOptions.color }"></div>
            <select id="color-select" v-model="exportOptions.color">
              <option value="#FFFFFF">白色</option>
              <option value="#8B7D6B">棕</option>
              <option value="#A8A8A8">灰</option>
              <option value="#D4B5A0">米</option>
              <option value="#C7B299">卡其</option>
              <option value="#B8A082">駝</option>
              <option value="#9B8B7A">深棕</option>
              <option value="#E8D5C4">淺米</option>
              <option value="#F5F1EB">奶白</option>
            </select>
          </div>
        </div>

        <div class="option-group">
          <label for="weight-select">粗細：</label>
          <select id="weight-select" v-model="exportOptions.weight">
            <option value="normal">正常</option>
            <option value="bold">粗體</option>
            <option value="lighter">細體</option>
          </select>
        </div>

        <div class="option-group">
          <label for="background-select">背景：</label>
          <div class="background-preview-container">
            <div class="background-preview" :style="backgroundPreviewStyle"></div>
            <select id="background-select" v-model="exportOptions.background">
              <option value="transparent">透明</option>
              <option value="#FFFFFF">白色</option>
              <option value="#F5F1EB">奶白</option>
              <option value="#E8D5C4">淺米</option>
              <option value="#D4B5A0">米色</option>
              <option value="#C7B299">卡其</option>
              <option value="#B8A082">駝色</option>
              <option value="#A8A8A8">灰色</option>
              <option value="#8B7D6B">棕色</option>
            </select>
          </div>
        </div>

        <div class="modal-buttons">
          <button @click="showExportModal = false" class="cancel-btn">取消</button>
          <button @click="confirmExport" class="confirm-btn">確認輸出</button>
        </div>
      </div>
    </div>
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
  min-height: 100vh;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Microsoft JhengHei', 'PingFang TC', 'Helvetica Neue', Arial, sans-serif;
  position: relative;
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
  flex-shrink: 1;
}

@media (max-width: 768px) {
  .app {
    height: 100vh;
    height: 100dvh; /* 使用動態視窗高度 */
  }
  
  .header {
    flex-direction: row;
    gap: 0.5rem;
    text-align: left;
    flex-shrink: 0;
    align-items: center;
  }
  
  .header h1 {
    flex: 1;
    text-align: center;
    font-size: 1.2rem;
  }
  
  .export-btn {
    font-size: 0.8rem;
    padding: 0.4rem 0.8rem;
  }
  
  .main-content {
    flex: 1;
    min-height: 0;
    overflow-y: auto;
  }
}

/* 彈出窗口樣式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  max-width: 400px;
  width: 90%;
}

.modal-content h3 {
  margin: 0 0 1.5rem 0;
  color: #333;
  text-align: center;
}

.option-group {
  margin-bottom: 1rem;
}

.option-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
  color: #333;
}

.color-preview-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.color-preview {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  border: 1px solid #ddd;
  flex-shrink: 0;
}

.background-preview-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.background-preview {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  border: 1px solid #ddd;
  flex-shrink: 0;
  background-size: 8px 8px;
}

.option-group select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

.modal-buttons {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
}

.cancel-btn, .confirm-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
}

.cancel-btn {
  background: #f5f5f5;
  color: #333;
}

.cancel-btn:hover {
  background: #e0e0e0;
}

.confirm-btn {
  background: #667eea;
  color: white;
}

.confirm-btn:hover {
  background: #5a6fd8;
}
</style>
