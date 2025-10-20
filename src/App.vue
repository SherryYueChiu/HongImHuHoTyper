<script setup lang="ts">
import { ref } from 'vue'
import VirtualKeyboard from '@/components/VirtualKeyboard.vue'
import ResultDisplay from '@/components/ResultDisplay.vue'
import InputBar from '@/components/InputBar.vue'
import ExportPanel from '@/components/ExportPanel.vue'

// 主要狀態
const inputText = ref('')
const resultText = ref('')
const showResult = ref(false)
const inputBarRef = ref<InstanceType<typeof InputBar>>()
const cursorPosition = ref(0)

// 輸出選項（改由 ExportPanel 處理），此處僅保留開關與字體
const showExportModal = ref(false)
const exportFontFamily = ref('jf-openhuninn')
const textAlignment = ref<'left'|'center'|'right'>('left')
const lineSpacing = ref(1.0)

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

// 匯出由 ExportPanel 處理

// 背景預覽邏輯已抽離到 ExportPanel

// 字體轉換邏輯由子組件處理

// 匯出功能已抽離
</script>

<template>
  <div class="app">
    <!-- 標題 -->
    <header class="header">
      <h1>台語方音符號輸入法</h1>
      <button @click="showExportOptions" class="export-btn">
        更多
      </button>
    </header>

    <!-- 上半部：結果展示區 -->
    <main class="main-content">
      <ResultDisplay 
        :text="resultText" 
        :show="showResult"
        :font-family="exportFontFamily"
        :alignment="textAlignment"
        :line-spacing="lineSpacing"
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

    <ExportPanel 
      v-model="showExportModal"
      v-model:fontFamily="exportFontFamily"
      v-model:alignment="textAlignment"
      v-model:lineSpacing="lineSpacing"
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
