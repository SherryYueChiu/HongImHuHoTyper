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
  weight: 'bold',
  background: false,
  backgroundColor: '#FFFFFF',
  backgroundOpacity: 1,
  stroke: false,
  strokeColor: '#000000',
  strokeWidth: 1,
  fontFamily: 'jf-openhuninn',
  rounded: false,
  borderRadius: 10,
  borderColor: '#FFFFFF'
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
  if (!exportOptions.value.background) {
    return {
      backgroundColor: 'transparent',
      backgroundImage: 'url("data:image/svg+xml,%3Csvg xmlns=\'http://www.w3.org/2000/svg\' width=\'20\' height=\'20\' viewBox=\'0 0 20 20\'%3E%3Cg fill=\'%23f0f0f0\'%3E%3Crect x=\'0\' y=\'0\' width=\'10\' height=\'10\'/%3E%3Crect x=\'10\' y=\'10\' width=\'10\' height=\'10\'/%3E%3C/g%3E%3C/svg%3E")'
    }
  }
  const color = exportOptions.value.backgroundColor
  const opacity = exportOptions.value.backgroundOpacity
  return {
    backgroundColor: `rgba(${parseInt(color.slice(1, 3), 16)}, ${parseInt(color.slice(3, 5), 16)}, ${parseInt(color.slice(5, 7), 16)}, ${opacity})`,
    backgroundImage: 'none'
  }
})

// 獲取字體名稱
const getFontFamily = (fontFamily: string) => {
  const fontMap: { [key: string]: string } = {
    'default': '"Microsoft JhengHei", "PingFang TC", sans-serif',
    'cjkFonts': '"cjkFonts", "Microsoft JhengHei", sans-serif',
    'jf-openhuninn': '"jf-openhuninn", "Microsoft JhengHei", sans-serif'
  }
  return fontMap[fontFamily] || fontMap['default']
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
  
  // 提高畫質 - 設置高DPI
  const devicePixelRatio = window.devicePixelRatio || 1
  const scaleFactor = 2 // 2倍解析度
  
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
  
  
  // 設置高解析度 canvas
  const baseWidth = contentWidth + padding * 2
  const baseHeight = contentHeight + padding * 2
  
  canvas.width = baseWidth * scaleFactor
  canvas.height = baseHeight * scaleFactor
  canvas.style.width = baseWidth + 'px'
  canvas.style.height = baseHeight + 'px'
  
  // 縮放 context 以匹配高解析度
  ctx.scale(scaleFactor, scaleFactor)
  
  // 設置背景和圓角邊框
  if (exportOptions.value.background || exportOptions.value.rounded) {
    const radius = exportOptions.value.rounded ? exportOptions.value.borderRadius : 0
    const borderWidth = exportOptions.value.rounded ? 2 : 0
    const borderPadding = borderWidth / 2
    
    // 計算背景和邊框的實際區域
    const bgX = borderPadding
    const bgY = borderPadding
    const bgWidth = baseWidth - borderWidth
    const bgHeight = baseHeight - borderWidth
    
    // 繪製背景（只在圓角區域內）
    if (exportOptions.value.background) {
      const color = exportOptions.value.backgroundColor
      const opacity = exportOptions.value.backgroundOpacity
      const r = parseInt(color.slice(1, 3), 16)
      const g = parseInt(color.slice(3, 5), 16)
      const b = parseInt(color.slice(5, 7), 16)
      ctx.fillStyle = `rgba(${r}, ${g}, ${b}, ${opacity})`
      
      if (exportOptions.value.rounded) {
        ctx.beginPath()
        ctx.roundRect(bgX, bgY, bgWidth, bgHeight, radius)
        ctx.fill()
      } else {
        ctx.fillRect(bgX, bgY, bgWidth, bgHeight)
      }
    }
    
    // 繪製圓角邊框
    if (exportOptions.value.rounded) {
      const borderColor = exportOptions.value.borderColor
      ctx.strokeStyle = borderColor
      ctx.lineWidth = borderWidth
      ctx.beginPath()
      ctx.roundRect(bgX, bgY, bgWidth, bgHeight, radius)
      ctx.stroke()
    }
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
      
      const fontWeight = exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal'
      const fontFamily = getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} 32px ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      
      // 繪製描邊
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = exportOptions.value.strokeColor
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(chineseChar.textContent || '', charX, charY)
      }
      
      // 繪製填充
      ctx.fillStyle = exportOptions.value.color
      ctx.fillText(chineseChar.textContent || '', charX, charY)
    }
    
    // 繪製注音符號
    const phoneticSymbols = group.querySelectorAll('.phonetic-symbol')
    phoneticSymbols.forEach((symbol, index) => {
      const symbolRect = symbol.getBoundingClientRect()
      const symbolX = symbolRect.left - containerRect.left - minX + padding
      const symbolY = symbolRect.top - containerRect.top - minY + padding
      
      const fontWeight = exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal'
      const fontFamily = getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} 18px ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      
      // 增加注音符號之間的間距
      const spacing = index * 2
      const symbolYPos = symbolY + spacing
      
      // 繪製描邊
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = exportOptions.value.strokeColor
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(symbol.textContent || '', symbolX, symbolYPos)
      }
      
      // 繪製填充
      ctx.fillStyle = exportOptions.value.color
      ctx.fillText(symbol.textContent || '', symbolX, symbolYPos)
    })
    
    // 繪製音調標註
    const toneMarks = group.querySelectorAll('.tone-mark')
    toneMarks.forEach((tone) => {
      if (tone.classList.contains('transparent')) return
      
      const toneRect = tone.getBoundingClientRect()
      const toneX = toneRect.left - containerRect.left - minX + padding + 3
      const toneY = toneRect.top - containerRect.top - minY + padding
      
      // 調整音調位置，進一步降低高度
      const fontWeight = exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal'
      const fontFamily = getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} 12px ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      
      // 繪製描邊
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = exportOptions.value.strokeColor
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(tone.textContent || '', toneX, toneY)
      }
      
      // 繪製填充
      ctx.fillStyle = exportOptions.value.color
      ctx.fillText(tone.textContent || '', toneX, toneY)
    })
    
    // 繪製 plain text
    const plainTextChar = group.querySelector('.plain-text-char')
    if (plainTextChar) {
      const textRect = plainTextChar.getBoundingClientRect()
      const textX = textRect.left - containerRect.left - minX + padding
      const textY = textRect.top - containerRect.top - minY + padding
      
      // 調整 plain text 位置，提高高度
      const fontWeight = exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal'
      const fontFamily = getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} 16px ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      
      // 繪製描邊
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = exportOptions.value.strokeColor
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(plainTextChar.textContent || '', textX, textY)
      }
      
      // 繪製填充
      ctx.fillStyle = exportOptions.value.color
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
        :font-family="exportOptions.fontFamily"
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
          <label for="font-select">字體：</label>
          <select id="font-select" v-model="exportOptions.fontFamily">
            <option value="default">預設字體</option>
            <option value="cjkFonts">思源字體</option>
            <option value="jf-openhuninn">粉圓體</option>
          </select>
        </div>

        <div class="option-group">
          <label>
            <input type="checkbox" v-model="exportOptions.background" class="background-checkbox">
            背景
          </label>
        </div>

        <div v-if="exportOptions.background" class="option-group">
          <label for="background-color-select">背景顏色：</label>
          <div class="background-preview-container">
            <div class="background-preview" :style="backgroundPreviewStyle"></div>
            <select id="background-color-select" v-model="exportOptions.backgroundColor">
              <option value="#FFFFFF">白色</option>
              <option value="#F5F1EB">奶白</option>
              <option value="#E8D5C4">淺米</option>
              <option value="#D4B5A0">米色</option>
              <option value="#B8A082">駝色</option>
              <option value="#A8A8A8">灰色</option>
              <option value="#000000">黑色</option>
            </select>
          </div>
        </div>

        <div v-if="exportOptions.background" class="option-group">
          <label for="opacity-range">透明度：</label>
          <div class="opacity-container">
            <input 
              type="range" 
              id="opacity-range" 
              v-model="exportOptions.backgroundOpacity" 
              min="0" 
              max="1" 
              step="0.1"
              class="opacity-slider"
            >
            <span class="opacity-value">{{ Math.round(exportOptions.backgroundOpacity * 100) }}%</span>
          </div>
        </div>

        <div class="option-group">
          <label>
            <input type="checkbox" v-model="exportOptions.stroke" class="stroke-checkbox">
            文字描邊
          </label>
        </div>

        <div v-if="exportOptions.stroke" class="option-group">
          <label for="stroke-color-select">描邊顏色：</label>
          <div class="color-preview-container">
            <div class="color-preview" :style="{ backgroundColor: exportOptions.strokeColor }"></div>
            <select id="stroke-color-select" v-model="exportOptions.strokeColor">
              <option value="#000000">黑色</option>
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

        <div v-if="exportOptions.stroke" class="option-group">
          <label for="stroke-width-select">描邊粗細：</label>
          <select id="stroke-width-select" v-model="exportOptions.strokeWidth">
            <option :value="1">細</option>
            <option :value="2">中</option>
            <option :value="3">粗</option>
            <option :value="4">很粗</option>
          </select>
        </div>

        <div class="option-group">
          <label>
            <input type="checkbox" v-model="exportOptions.rounded" class="rounded-checkbox">
            圓角邊框
          </label>
        </div>

        <div v-if="exportOptions.rounded" class="option-group">
          <label for="border-radius-select">圓角大小：</label>
          <select id="border-radius-select" v-model="exportOptions.borderRadius">
            <option :value="5">小</option>
            <option :value="10">中</option>
            <option :value="15">大</option>
            <option :value="20">很大</option>
          </select>
        </div>

        <div v-if="exportOptions.rounded" class="option-group">
          <label for="border-color-select">邊框顏色：</label>
          <div class="color-preview-container">
            <div class="color-preview" :style="{ backgroundColor: exportOptions.borderColor }"></div>
            <select id="border-color-select" v-model="exportOptions.borderColor">
              <option value="#FFFFFF">白色</option>
              <option value="#000000">黑色</option>
              <option value="#8B7D6B">棕</option>
              <option value="#A8A8A8">灰</option>
              <option value="#D4B5A0">米</option>
              <option value="#B8A082">駝</option>
              <option value="#9B8B7A">深棕</option>
              <option value="#E8D5C4">淺米</option>
              <option value="#F5F1EB">奶白</option>
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

/* 字體定義 */
@font-face {
  font-family: 'cjkFonts';
  src: url('/fonts/cjkFonts_allseto_v1.11.ttf') format('truetype');
}

@font-face {
  font-family: 'jf-openhuninn';
  src: url('/fonts/jf-openhuninn_v2.1.ttf') format('truetype');
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
  max-height: 80vh;
  overflow-y: auto;
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

.stroke-checkbox, .rounded-checkbox, .background-checkbox {
  margin-right: 0.5rem;
}

.opacity-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.opacity-slider {
  flex: 1;
  height: 4px;
  border-radius: 2px;
  background: #ddd;
  outline: none;
  -webkit-appearance: none;
}

.opacity-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #667eea;
  cursor: pointer;
}

.opacity-slider::-moz-range-thumb {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #667eea;
  cursor: pointer;
  border: none;
}

.opacity-value {
  min-width: 40px;
  text-align: right;
  font-size: 0.9rem;
  color: #666;
}
</style>
