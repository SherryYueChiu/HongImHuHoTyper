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

// 清除內容
const clearContent = () => {
  inputText.value = ''
  resultText.value = ''
  showResult.value = false
}
</script>

<template>
  <div class="app">
    <!-- 標題 -->
    <header class="header">
      <h1>台語方音符號輸入法</h1>
    </header>

    <!-- 上半部：結果展示區 -->
    <main class="main-content">
      <ResultDisplay 
        :text="resultText" 
        :show="showResult"
        @clear="clearContent"
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
  text-align: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.header h1 {
  color: white;
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
}

.main-content {
  flex: 1;
  padding: 1rem;
  overflow-y: auto;
  min-height: 0;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 1.2rem;
  }
}
</style>
