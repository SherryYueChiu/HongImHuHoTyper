<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  text: string
  show: boolean
}

interface CharacterGroup {
  char: string
  phonetics: string[]
  tone: string[]
  isPlainText?: boolean
}

const props = defineProps<Props>()
const emit = defineEmits<{
  clear: []
}>()

// 處理文字，將注音符號和音調組合在一起
const processedText = computed((): CharacterGroup[] => {
  if (!props.text) return []
  
  const phoneticMarks = ['ㄅ', 'ㄆ', 'ㄇ', 'ㄈ', 'ㄉ', 'ㄊ', 'ㄋ', 'ㄌ', 'ㄍ', 'ㄎ', 'ㄏ', 
                        'ㄐ', 'ㄑ', 'ㄒ', 'ㄓ', 'ㄔ', 'ㄕ', 'ㄖ', 'ㄗ', 'ㄘ', 'ㄙ',
                        'ㄚ', 'ㄛ', 'ㄜ', 'ㄝ', 'ㄞ', 'ㄟ', 'ㄠ', 'ㄡ', 'ㄢ', 'ㄣ', 'ㄤ', 'ㄥ', 'ㄦ', 'ㄧ', 'ㄨ', 'ㄩ',
                        'ㆠ', 'ㆣ', 'ㆢ', 'ㆡ', 'ㆩ', 'ㆧ', 'ㆥ', 'ㆮ', 'ㆯ', 'ㆪ', 'ㆫ', 'ㆬ', 'ㆭ', 'ㆰ', 'ㆱ',
                        'ㆦ', 'ㆤ', 'ㆲ', 'ㄫ', 'ɤ']
  const toneMarks = ['ˊ', 'ˇ', 'ˋ', '˙', 'ˉ', '˪', '˫', 'ㆴ', 'ㆵ', 'ㆻ', 'ㆷ', '.']
  
  // 將文字分組：每個字符組包含一個漢字和其對應的注音符號+音調
  const groups = []
  let currentGroup = {
    char: '',
    phonetics: [] as string[],
    tone: [] as string[]
  }
  let plainTextBuffer = ''
  
  for (let i = 0; i < props.text.length; i++) {
    const char = props.text[i]
    if (!char) continue
    
    console.log('Processing char:', char, 'isPhonetic:', phoneticMarks.includes(char))
    
    if (phoneticMarks.includes(char)) {
      // 如果當前組有漢字，先保存當前組
      if (currentGroup.char) {
        groups.push({ ...currentGroup })
        currentGroup = { char: '', phonetics: [], tone: [] }
      }
      
      // 如果當前組有注音符號且已經有音調，先保存當前組
      if (currentGroup.phonetics.length > 0 && currentGroup.tone.length > 0) {
        groups.push({ ...currentGroup })
        currentGroup = { char: '', phonetics: [], tone: [] }
      }
      
      // 如果有累積的一般文字，先保存一般文字
      if (plainTextBuffer) {
        groups.push({ 
          char: plainTextBuffer, 
          phonetics: [], 
          tone: [], 
          isPlainText: true 
        })
        plainTextBuffer = ''
      }
      
      // 開始新的注音組
      currentGroup.phonetics.push(char)
    } else if (toneMarks.includes(char)) {
      currentGroup.tone.push(char)
    } else if (char === ' ') {
      // 空格表示一聲，設置音調為一聲
      currentGroup.tone.push('ˉ')
      // 保存當前組並開始新組
      if (currentGroup.char || currentGroup.phonetics.length > 0) {
        groups.push({ ...currentGroup })
      }
      currentGroup = { char: '', phonetics: [], tone: [] }
      // 跳過空格，不添加為字符組
    } else if (char === '\n') {
      // 如果有累積的一般文字，先保存
      if (plainTextBuffer) {
        groups.push({ 
          char: plainTextBuffer, 
          phonetics: [], 
          tone: [], 
          isPlainText: true 
        })
        plainTextBuffer = ''
      }
      // 換行符號
      if (currentGroup.char || currentGroup.phonetics.length > 0) {
        groups.push({ ...currentGroup })
      }
      groups.push({ char: char, phonetics: [], tone: [] })
        currentGroup = { char: '', phonetics: [], tone: [] }
    } else {
      // 判斷是否為一般文字（ASCII 字符：英文字母、數字、標點符號）
      // 或者中文字符（如：你、會、嗎等）
      // 或者標點符號（如：？、！、。等）
      const isPlainText = char.charCodeAt(0) < 128 || /[\u4e00-\u9fff]/.test(char) || /[？！。，、；：""''（）【】《》〈〉「」『』]/.test(char)
      
      if (isPlainText) {
        // 如果當前組有注音符號，先保存當前組
        if (currentGroup.phonetics.length > 0) {
          groups.push({ ...currentGroup })
          currentGroup = { char: '', phonetics: [], tone: [] }
        }
        
        // 如果當前組有漢字，先保存當前組
        if (currentGroup.char) {
          groups.push({ ...currentGroup })
          currentGroup = { char: '', phonetics: [], tone: [] }
        }
        
        // 一般文字，累積到一般文字緩衝區
        plainTextBuffer += char
      } else {
        // 漢字或其他字符，先保存一般文字緩衝區
        if (plainTextBuffer) {
          groups.push({ 
            char: plainTextBuffer, 
            phonetics: [], 
            tone: [], 
            isPlainText: true 
          })
          plainTextBuffer = ''
        }
        
        // 保存當前的注音組（如果有注音符號）
        if (currentGroup.phonetics.length > 0) {
          groups.push({ ...currentGroup })
          currentGroup = { char: '', phonetics: [], tone: [] }
        }
        
        // 開始新的組（漢字）
        currentGroup = { char: char, phonetics: [], tone: [] }
      }
    }
  }
  
  // 保存最後的一般文字緩衝區
  if (plainTextBuffer) {
    groups.push({ 
      char: plainTextBuffer, 
      phonetics: [], 
      tone: [], 
      isPlainText: true 
    })
  }
  
  // 保存最後一個組
  if (currentGroup.char || currentGroup.phonetics.length > 0) {
    groups.push(currentGroup)
  }
  
  return groups
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
  
  // 設定畫布大小，添加一些邊距
  const padding = 40
  canvas.width = containerRect.width + padding * 2
  canvas.height = containerRect.height + padding * 2
  
  // 設定透明背景（不填充背景色）
  
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
      
      ctx.font = '36px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(chineseChar.textContent || '', charX, charY + 36)
    }
    
    // 繪製注音符號
    const phoneticSymbols = group.querySelectorAll('.phonetic-symbol')
    phoneticSymbols.forEach((symbol) => {
      const symbolRect = symbol.getBoundingClientRect()
      const symbolX = symbolRect.left - containerRect.left + padding
      const symbolY = symbolRect.top - containerRect.top + padding
      
      ctx.font = '20px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(symbol.textContent || '', symbolX, symbolY + 20)
    })
    
    // 繪製音調標註
    const toneMarks = group.querySelectorAll('.tone-mark')
    toneMarks.forEach((tone) => {
      if (tone.classList.contains('transparent')) return
      
      const toneRect = tone.getBoundingClientRect()
      const toneX = toneRect.left - containerRect.left + padding
      let toneY = toneRect.top - containerRect.top + padding
      
      // 所有音調標記都向下調整
      toneY += 4
      
      // 調整以ㄧ結尾的音調標記位置
      if (tone.classList.contains('yi-ending')) {
        toneY += 8 // 對應 CSS 中的 translateY(1em) 約等於 16px
      }
      
      ctx.font = '14px "Microsoft JhengHei", "PingFang TC", sans-serif'
      ctx.fillStyle = '#333'
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      ctx.fillText(tone.textContent || '', toneX, toneY + 14)
    })
    
    // 繪製 plain text
    const plainTextChar = group.querySelector('.plain-text-char')
    if (plainTextChar) {
      const textRect = plainTextChar.getBoundingClientRect()
      const textX = textRect.left - containerRect.left + padding
      const textY = textRect.top - containerRect.top + padding
      
      ctx.font = '20px "Microsoft JhengHei", "PingFang TC", sans-serif'
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

const clearContent = () => {
  emit('clear')
}
</script>

<template>
  <div class="result-display">
    <div v-if="show" class="result-container">
      <div class="result-header">
        <h3>結果展示</h3>
        <div class="action-buttons">
          <button @click="exportImage" class="export-btn">
            📷 輸出圖片
          </button>
          <button @click="clearContent" class="clear-btn">
            🗑️ 清除
          </button>
        </div>
      </div>
      
      <div class="result-content">
         <!-- 直式書寫顯示 -->
         <div class="vertical-display">
           <div 
             v-for="(item, index) in processedText" 
             :key="index" 
             :class="[
               'character-group', 
               { 
                 'plain-text': item.isPlainText,
                 'line-break': item.char === '\n'
               }
             ]"
           >
             <!-- 一般文字 -->
             <span v-if="item.isPlainText" class="plain-text-char">
               {{ item.char }}
             </span>
             
             <!-- 漢字 -->
             <div v-else-if="item.char" class="chinese-char">
               {{ item.char }}
             </div>
             
             <!-- 注音符號（直式排列） -->
             <div v-if="item.phonetics.length > 0" class="phonetic-column">
               <div 
                 v-for="(phonetic, phoneticIndex) in item.phonetics" 
                 :key="phoneticIndex"
                 :class="[
                   'phonetic-symbol', 
                   { 
                     'short-yi': phonetic === 'ㄧ',
                     'short-wu-yu': phonetic === 'ㄨ' || phonetic === 'ㄩ'
                   }
                 ]"
               >
                 {{ phonetic }}
               </div>
               <!-- 音調標註（在最下面符號的右邊角落，橫向排列） -->
               <div 
                 v-if="item.tone.length > 0" 
                 class="tone-marks"
               >
                 <span 
                   v-for="(tone, toneIndex) in item.tone" 
                   :key="toneIndex"
                   :class="[
                     'tone-mark', 
                     { 
                       'transparent': tone === 'ˉ',
                       'special-tone': ['ㆴ', 'ㆵ', 'ㆻ', 'ㆷ'].includes(tone),
                       'yi-ending': item.phonetics.length > 0 && item.phonetics[item.phonetics.length - 1] === 'ㄧ'
                     }
                   ]"
                 >
                   {{ tone }}
                 </span>
               </div>
             </div>
           </div>
         </div>
      </div>
    </div>
    
    <div v-else class="empty-state">
      <p>請輸入文字或使用下方虛擬鍵盤</p>
    </div>
  </div>
</template>

<style scoped>
.result-display {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.result-container {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  height: 100%;
  display: flex;
  flex-direction: column;
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e0e0e0;
}

.result-header h3 {
  margin: 0;
  color: #333;
  font-size: 1.2rem;
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
}

.export-btn, .clear-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s ease;
}

.export-btn {
  background: #4CAF50;
  color: white;
}

.export-btn:hover {
  background: #45a049;
  transform: translateY(-1px);
}

.clear-btn {
  background: #f44336;
  color: white;
}

.clear-btn:hover {
  background: #da190b;
  transform: translateY(-1px);
}

.result-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.vertical-display {
  flex: 1;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  border: 1px solid rgba(103, 126, 234, 0.3);
  min-height: 200px;
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  gap: .2em;
  overflow-y: auto;
  line-height: 1.5;
}

.character-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0.2rem;
  margin-right: .3em;
  position: relative;
  justify-content: center;
  align-self: center;
  height: auto;
}

.character-group.plain-text {
  display: inline;
  margin: 0;
  align-self: center;
  vertical-align: middle;
}

.character-group.line-break {
  width: 100%;
  height: 1.5em;
  display: block;
  margin: 0;
  align-self: stretch;
}

.plain-text-char {
  font-size: 1rem;
  color: #333;
  font-family: 'Microsoft JhengHei', 'PingFang TC', sans-serif;
  display: inline;
}

.chinese-char {
  font-size: 1.5rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 0.3rem;
  font-family: 'Microsoft JhengHei', 'PingFang TC', sans-serif;
}

.phonetic-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  justify-content: flex-start;
}

.phonetic-symbol {
  font-size: 1.2rem;
  color: #333;
  font-weight: 500;
  margin: 0;
  text-align: center;
  font-family: 'Microsoft JhengHei', 'PingFang TC', sans-serif;
  line-height: 1;
}

.phonetic-symbol.short-yi {
  margin-top: -0.3em;
  margin-bottom: -0.4em;
}

.phonetic-symbol.short-wu-yu {
  margin-top: -0.1em;
  margin-bottom: -0.1em;
}

.tone-marks {
  position: absolute;
  bottom: .6em;
  left: 1em;
  display: flex;
  flex-direction: row;
  gap: 0px;
  align-items: flex-start;
  z-index: 10;
}

.tone-mark {
  font-size: 0.75rem;
  color: #333;
  font-weight: bold;
  line-height: 1;
  white-space: nowrap;
  display: inline-block;
  margin-right: -.2em;
}

.tone-mark.transparent {
  color: transparent;
  width: 0;
  height: 0;
  margin: 0;
  padding: 0;
  font-size: 0;
  line-height: 0;
  display: none;
}

.tone-mark.special-tone {
  margin-right: -0.8em;
}

.tone-mark.yi-ending {
  transform: translateY(1em);
}


.empty-state {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.1rem;
}

@media (max-width: 768px) {
  .result-container {
    padding: 1rem;
  }
  
  .result-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .action-buttons {
    justify-content: center;
  }
  
  .horizontal-text {
    font-size: 1.2rem;
  }
  
  .phonetic-symbol {
    font-size: 1rem;
  }
  
  .phonetic-column {
    min-height: 60px;
    padding: 0.3rem;
  }
}
</style>
