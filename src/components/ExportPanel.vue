<script setup lang="ts">
import { ref, computed } from 'vue'
import ColorPicker from '@/components/ColorPicker.vue'

interface Props {
  modelValue: boolean
  fontFamily?: string
  alignment?: 'left' | 'center' | 'right'
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
  (e: 'update:fontFamily', value: string): void
  (e: 'update:alignment', value: 'left' | 'center' | 'right'): void
}>()

const exportOptions = ref({
  color: '#000000',
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
  borderColor: '#FFFFFF',
  borderEnabled: false,
  borderWidth: 1,
  // 新增
  alignment: 'left' as 'left'|'center'|'right',
  shadow: false,
  shadowColor: '#000000',
  shadowBlur: 0,
  shadowOffsetX: 0,
  shadowOffsetY: 0,
  // 顏色透明度
  textOpacity: 1,
  strokeOpacity: 1,
  borderOpacity: 1,
  shadowOpacity: 1,
  // 內留白（框內與文字之間的距離 / 匯出 padding）
  padding: 10
})

// 與父層同步字體
const computedFontFamily = computed({
  get: () => props.fontFamily ?? exportOptions.value.fontFamily,
  set: (val: string) => {
    exportOptions.value.fontFamily = val
    emit('update:fontFamily', val)
  }
})

const computedAlignment = computed({
  get: () => props.alignment ?? exportOptions.value.alignment,
  set: (val: 'left'|'center'|'right') => {
    exportOptions.value.alignment = val
    emit('update:alignment', val)
  }
})

// 側邊選單的當前項目
const activeSetting = ref<'font'|'color'|'weight'|'align'|'stroke'|'border'|'shadow'>('weight')

// 產生側欄圖示（以字串回傳供 v-html）
const getIconSvg = (icon: string) => {
  const map: Record<string, string> = {
    color: `<svg width="14" height="14" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><circle cx="12" cy="12" r="9" fill="currentColor" opacity=".15"/><path d="M12 3a9 9 0 100 18 9 9 0 000-18z" stroke="currentColor" stroke-width="1.5" fill="none"/></svg>`,
    weight: `<svg width="14" height="14" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><text x="6" y="18" font-size="14" font-weight="900" fill="currentColor">A</text></svg>`,
    align: `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M4 6h16M8 12h8M6 18h12" stroke="currentColor" stroke-width="1.5"/></svg>`,
    background: `<svg width="14" height="14" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><rect x="4" y="5" width="16" height="14" rx="2" fill="#000"/><text x="9" y="17" font-size="12" font-weight="800" fill="#fff">A</text></svg>`,
    stroke: `<svg width="14" height="14" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><text x="6" y="18" font-size="14" font-weight="800" fill="none" stroke="currentColor" stroke-width="1.6">A</text></svg>`,
    rounded: `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="5" y="7" width="14" height="10" rx="3" stroke="currentColor" stroke-width="1.5"/></svg>`,
    border: `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="6" y="8" width="12" height="8" rx="2" stroke="currentColor" stroke-width="1.5"/></svg>`,
    shadow: `<svg width="14" height="14" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><text x="7" y="17" font-size="14" font-weight="800" fill="currentColor" opacity="0.3" transform="translate(2,2)">A</text><text x="6" y="18" font-size="14" font-weight="800" fill="currentColor">A</text></svg>`
  }
  return map[icon] || ''
}

// 右側開關的代理值，讓 v-model 綁定合法成員表達式
const settingEnabled = computed<boolean>({
  get: () => {
    if (activeSetting.value === 'stroke') return exportOptions.value.stroke
    if (activeSetting.value === 'border') return exportOptions.value.borderEnabled
    if (activeSetting.value === 'shadow') return exportOptions.value.shadow
    return false
  },
  set: (val: boolean) => {
    if (activeSetting.value === 'stroke') exportOptions.value.stroke = val
    else if (activeSetting.value === 'border') exportOptions.value.borderEnabled = val
    else if (activeSetting.value === 'shadow') exportOptions.value.shadow = val
  }
})


const hexToRgba = (hex: string, alpha: number) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return `rgba(${r}, ${g}, ${b}, ${alpha})`
}

// 預覽樣式
const previewBoxStyle = computed(() => {
  const styles: Record<string, string> = {
    background: 'transparent',
    border: 'none',
    borderRadius: '0px'
  }
  // 背景色現在與邊框開關聯動
  if (exportOptions.value.borderEnabled) {
    const color = exportOptions.value.backgroundColor
    const opacity = exportOptions.value.backgroundOpacity
    styles.background = hexToRgba(color, opacity)
    styles.border = `${exportOptions.value.borderWidth}px solid ${hexToRgba(exportOptions.value.borderColor, exportOptions.value.borderOpacity)}`
    styles.borderRadius = `${exportOptions.value.borderRadius}px`
  } else {
    styles.borderRadius = `${exportOptions.value.borderRadius}px`
  }
  styles.padding = `${exportOptions.value.padding}px`
  return styles
})

const previewTextStyle = computed(() => {
  const fontMap: { [key: string]: string } = {
    'default': '"Microsoft JhengHei", "PingFang TC", sans-serif',
    'cjkFonts': '"cjkFonts", "Microsoft JhengHei", sans-serif',
    'jf-openhuninn': '"jf-openhuninn", "Microsoft JhengHei", sans-serif'
  }
  const weight = exportOptions.value.weight === 'bold' ? '900' : exportOptions.value.weight === 'lighter' ? '300' : '600'
  return {
    color: hexToRgba(exportOptions.value.color, exportOptions.value.textOpacity),
    fontFamily: fontMap[exportOptions.value.fontFamily] || fontMap['default'],
    fontWeight: weight as any,
    WebkitTextStroke: exportOptions.value.stroke ? `${exportOptions.value.strokeWidth}px ${hexToRgba(exportOptions.value.strokeColor, exportOptions.value.strokeOpacity)}` : '0 transparent',
    textShadow: exportOptions.value.shadow ? `${exportOptions.value.shadowOffsetX}px ${exportOptions.value.shadowOffsetY}px ${exportOptions.value.shadowBlur}px ${hexToRgba(exportOptions.value.shadowColor, exportOptions.value.shadowOpacity)}` : 'none'
  } as any
})

const getFontFamily = (fontFamily: string) => {
  const fontMap: { [key: string]: string } = {
    'default': '"Microsoft JhengHei", "PingFang TC", sans-serif',
    'cjkFonts': '"cjkFonts", "Microsoft JhengHei", sans-serif',
    'jf-openhuninn': '"jf-openhuninn", "Microsoft JhengHei", sans-serif'
  }
  return fontMap[fontFamily] || fontMap['default']
}

const close = () => emit('update:modelValue', false)

const confirmExport = () => {
  emit('update:modelValue', false)
  exportImage()
}

const exportImage = () => {
  const resultContainer = document.querySelector('.result-container')
  if (!resultContainer) return

  const containerRect = resultContainer.getBoundingClientRect()

  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')!

  const scaleFactor = 2

  // 更精準：量測所有子元素（漢字、注音、音調、純文字）
  const measureNodes = resultContainer.querySelectorAll('.chinese-char, .phonetic-symbol, .tone-mark:not(.transparent), .plain-text-char')
  let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity
  if (measureNodes.length > 0) {
    measureNodes.forEach((el) => {
      const node = el as HTMLElement
      const rect = node.getBoundingClientRect()
      const relativeX = rect.left - containerRect.left
      const relativeY = rect.top - containerRect.top
      minX = Math.min(minX, relativeX)
      minY = Math.min(minY, relativeY)
      maxX = Math.max(maxX, relativeX + rect.width)
      maxY = Math.max(maxY, relativeY + rect.height)
    })
  } else {
    const groups = resultContainer.querySelectorAll('.character-group')
    groups.forEach((group) => {
      if ((group as HTMLElement).classList.contains('line-break')) return
      const rect = (group as HTMLElement).getBoundingClientRect()
      const relativeX = rect.left - containerRect.left
      const relativeY = rect.top - containerRect.top
      minX = Math.min(minX, relativeX)
      minY = Math.min(minY, relativeY)
      maxX = Math.max(maxX, relativeX + rect.width)
      maxY = Math.max(maxY, relativeY + rect.height)
    })
  }

  // 防呆：若完全沒有量測到內容，使用容器尺寸避免 NaN 導致空白輸出
  if (!isFinite(minX) || !isFinite(minY) || !isFinite(maxX) || !isFinite(maxY)) {
    minX = 0
    minY = 0
    maxX = containerRect.width
    maxY = containerRect.height
  }

  const padding = exportOptions.value.padding
  const contentWidth = Math.max(0, maxX - minX)
  const contentHeight = Math.max(0, maxY - minY)

  // 擴張區域，避免描邊與陰影讓內容貼齊邊緣
  const strokeExpand = exportOptions.value.stroke ? Math.ceil(exportOptions.value.strokeWidth / 2) : 0
  const shadowExpandX = exportOptions.value.shadow ? Math.ceil(Math.max(0, Math.abs(exportOptions.value.shadowOffsetX)) + exportOptions.value.shadowBlur) : 0
  const shadowExpandY = exportOptions.value.shadow ? Math.ceil(Math.max(0, Math.abs(exportOptions.value.shadowOffsetY)) + exportOptions.value.shadowBlur) : 0
  const expandX = Math.max(strokeExpand, shadowExpandX)
  const expandY = Math.max(strokeExpand, shadowExpandY)

  // 計算最終的 canvas 尺寸，確保包含所有必要的區域
  const finalWidth = contentWidth + padding * 2 + expandX * 2
  const finalHeight = contentHeight + padding * 2 + expandY * 2

  canvas.width = finalWidth * scaleFactor
  canvas.height = finalHeight * scaleFactor
  canvas.style.width = finalWidth + 'px'
  canvas.style.height = finalHeight + 'px'

  ctx.scale(scaleFactor, scaleFactor)
  // 使用 translate 來處理 padding 和擴張量
  ctx.save()
  ctx.translate(padding + expandX, padding + expandY)

  // helpers
  const drawRoundRect = (ctx2: CanvasRenderingContext2D, x: number, y: number, w: number, h: number, r: number) => {
    const radius = Math.max(0, r)
    const rr = Math.min(radius, w / 2, h / 2)
    ctx2.beginPath()
    ctx2.moveTo(x + rr, y)
    ctx2.lineTo(x + w - rr, y)
    ctx2.quadraticCurveTo(x + w, y, x + w, y + rr)
    ctx2.lineTo(x + w, y + h - rr)
    ctx2.quadraticCurveTo(x + w, y + h, x + w - rr, y + h)
    ctx2.lineTo(x + rr, y + h)
    ctx2.quadraticCurveTo(x, y + h, x, y + h - rr)
    ctx2.lineTo(x, y + rr)
    ctx2.quadraticCurveTo(x, y, x + rr, y)
    ctx2.closePath()
  }

  // 背景與邊框（含圓角）：以目前平移後的原點為基準（內容左上 = 0,0）且包含 padding 區域
  if (exportOptions.value.borderEnabled || exportOptions.value.borderRadius > 0) {
    const radius = exportOptions.value.borderRadius
    const borderWidth = exportOptions.value.borderEnabled ? exportOptions.value.borderWidth : 0

    // 背景和邊框的繪製區域應該包含 padding 和擴張區域
    const bgX = -padding - expandX
    const bgY = -padding - expandY
    const bgWidth = finalWidth
    const bgHeight = finalHeight

    // 背景色現在與邊框開關聯動
    if (exportOptions.value.borderEnabled) {
      ctx.fillStyle = hexToRgba(exportOptions.value.backgroundColor, exportOptions.value.backgroundOpacity) as any
      drawRoundRect(ctx, bgX, bgY, bgWidth, bgHeight, radius)
      ctx.fill()
    }

    if (exportOptions.value.borderEnabled) {
      ctx.strokeStyle = hexToRgba(exportOptions.value.borderColor, exportOptions.value.borderOpacity) as any
      ctx.lineWidth = borderWidth
      drawRoundRect(ctx, bgX, bgY, bgWidth, bgHeight, radius)
      ctx.stroke()
    }
  }

  const characterGroups = resultContainer.querySelectorAll('.character-group')
  characterGroups.forEach((group) => {
    if ((group as HTMLElement).classList.contains('line-break')) return
    const groupRect = (group as HTMLElement).getBoundingClientRect()
    const relativeX = groupRect.left - containerRect.left - minX
    const relativeY = groupRect.top - containerRect.top - minY

    const chineseChar = (group as HTMLElement).querySelector('.chinese-char') as HTMLElement | null
    if (chineseChar) {
      const charRect = chineseChar.getBoundingClientRect()
      const charX = charRect.left - containerRect.left - minX
      const charY = charRect.top - containerRect.top - minY
      const cs = window.getComputedStyle(chineseChar)
      const fontWeight = cs.fontWeight || (exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal')
      const fontSize = cs.fontSize || '32px'
      const fontFamily = cs.fontFamily || getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} ${fontSize} ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 陰影
      if (exportOptions.value.shadow) {
        ctx.shadowColor = hexToRgba(exportOptions.value.shadowColor, exportOptions.value.shadowOpacity) as any
        ctx.shadowBlur = exportOptions.value.shadowBlur
        ctx.shadowOffsetX = exportOptions.value.shadowOffsetX
        ctx.shadowOffsetY = exportOptions.value.shadowOffsetY
      } else {
        ctx.shadowColor = 'rgba(0,0,0,0)'
        ctx.shadowBlur = 0
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
      }
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = hexToRgba(exportOptions.value.strokeColor, exportOptions.value.strokeOpacity) as any
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(chineseChar.textContent || '', charX, charY)
      }
      ctx.fillStyle = hexToRgba(exportOptions.value.color, exportOptions.value.textOpacity) as any
      ctx.fillText(chineseChar.textContent || '', charX, charY)
    }

    const phoneticSymbols = (group as HTMLElement).querySelectorAll('.phonetic-symbol')
    ;(Array.from(phoneticSymbols) as HTMLElement[]).forEach((symbol) => {
      const symbolRect = symbol.getBoundingClientRect()
      const symbolX = symbolRect.left - containerRect.left - minX
      const symbolY = symbolRect.top - containerRect.top - minY
      const cs = window.getComputedStyle(symbol)
      const fontWeight = cs.fontWeight || (exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal')
      const fontSize = cs.fontSize || '18px'
      const fontFamily = cs.fontFamily || getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} ${fontSize} ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 陰影
      if (exportOptions.value.shadow) {
        ctx.shadowColor = hexToRgba(exportOptions.value.shadowColor, exportOptions.value.shadowOpacity) as any
        ctx.shadowBlur = exportOptions.value.shadowBlur
        ctx.shadowOffsetX = exportOptions.value.shadowOffsetX
        ctx.shadowOffsetY = exportOptions.value.shadowOffsetY
      } else {
        ctx.shadowColor = 'rgba(0,0,0,0)'
        ctx.shadowBlur = 0
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
      }
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = hexToRgba(exportOptions.value.strokeColor, exportOptions.value.strokeOpacity) as any
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(symbol.textContent || '', symbolX, symbolY)
      }
      ctx.fillStyle = hexToRgba(exportOptions.value.color, exportOptions.value.textOpacity) as any
      ctx.fillText(symbol.textContent || '', symbolX, symbolY)
    })

    const toneMarks = (group as HTMLElement).querySelectorAll('.tone-mark')
    ;(Array.from(toneMarks) as HTMLElement[]).forEach((tone) => {
      if (tone.classList.contains('transparent')) return
      const toneRect = tone.getBoundingClientRect()
      const toneX = toneRect.left - containerRect.left - minX
      const toneY = toneRect.top - containerRect.top - minY
      const cs = window.getComputedStyle(tone)
      const fontWeight = cs.fontWeight || (exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal')
      const fontSize = cs.fontSize || '12px'
      const fontFamily = cs.fontFamily || getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} ${fontSize} ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 陰影
      if (exportOptions.value.shadow) {
        ctx.shadowColor = hexToRgba(exportOptions.value.shadowColor, exportOptions.value.shadowOpacity) as any
        ctx.shadowBlur = exportOptions.value.shadowBlur
        ctx.shadowOffsetX = exportOptions.value.shadowOffsetX
        ctx.shadowOffsetY = exportOptions.value.shadowOffsetY
      } else {
        ctx.shadowColor = 'rgba(0,0,0,0)'
        ctx.shadowBlur = 0
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
      }
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = hexToRgba(exportOptions.value.strokeColor, exportOptions.value.strokeOpacity) as any
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(tone.textContent || '', toneX, toneY)
      }
      ctx.fillStyle = hexToRgba(exportOptions.value.color, exportOptions.value.textOpacity) as any
      ctx.fillText(tone.textContent || '', toneX, toneY)
    })

    const plainTextChar = (group as HTMLElement).querySelector('.plain-text-char') as HTMLElement | null
    if (plainTextChar) {
      const textRect = plainTextChar.getBoundingClientRect()
      const textX = textRect.left - containerRect.left - minX
      const textY = textRect.top - containerRect.top - minY
      const cs = window.getComputedStyle(plainTextChar)
      const fontWeight = cs.fontWeight || (exportOptions.value.weight === 'bold' ? 'bold' : exportOptions.value.weight === 'lighter' ? 'lighter' : 'normal')
      const fontSize = cs.fontSize || '16px'
      const fontFamily = cs.fontFamily || getFontFamily(exportOptions.value.fontFamily)
      ctx.font = `${fontWeight} ${fontSize} ${fontFamily}`
      ctx.textAlign = 'left'
      ctx.textBaseline = 'top'
      // 陰影
      if (exportOptions.value.shadow) {
        ctx.shadowColor = hexToRgba(exportOptions.value.shadowColor, exportOptions.value.shadowOpacity) as any
        ctx.shadowBlur = exportOptions.value.shadowBlur
        ctx.shadowOffsetX = exportOptions.value.shadowOffsetX
        ctx.shadowOffsetY = exportOptions.value.shadowOffsetY
      } else {
        ctx.shadowColor = 'rgba(0,0,0,0)'
        ctx.shadowBlur = 0
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
      }
      if (exportOptions.value.stroke) {
        ctx.strokeStyle = hexToRgba(exportOptions.value.strokeColor, exportOptions.value.strokeOpacity) as any
        ctx.lineWidth = exportOptions.value.strokeWidth
        ctx.strokeText(plainTextChar.textContent || '', textX, textY)
      }
      ctx.fillStyle = hexToRgba(exportOptions.value.color, exportOptions.value.textOpacity) as any
      ctx.fillText(plainTextChar.textContent || '', textX, textY)
    }
  })

  // 還原平移
  ctx.restore()

  const link = document.createElement('a')
  link.download = '玥餅啾咪.png'
  link.href = canvas.toDataURL('image/png')
  link.click()
}
</script>

<template>
  <div v-if="props.modelValue" class="modal-overlay" @click="close">
    <div class="modal-content" @click.stop>
      <h3>更多設定</h3>

      <div>
        <div class="export-layout">
          <aside class="sidebar">
               <button 
                 v-for="item in [
                   {key: 'weight', label: '粗細', icon: 'weight'},
                   {key: 'align', label: '對齊', icon: 'align'},
                   {key: 'stroke', label: '描邊', icon: 'stroke'},
                   {key: 'border', label: '邊框', icon: 'border'},
                   {key: 'shadow', label: '陰影', icon: 'shadow'}
                 ]"
              :key="item.key"
              :class="['sidebar-item', { active: activeSetting === (item.key as any) }]"
              @click="activeSetting = item.key as any"
            >
              <span class="icon" v-html="getIconSvg(item.icon)"></span>
              <span class="name">{{ item.label }}</span>
            </button>
          </aside>

          <section class="main">
            <div class="main-header">
              <div v-if="['stroke','border','shadow'].includes(activeSetting)" class="switch-row">
                <span>啟用</span>
                <label class="switch">
                  <input type="checkbox" v-model="settingEnabled" >
                  <span class="slider"></span>
                </label>
              </div>
            </div>

            <div class="main-body">
              <div v-if="activeSetting === 'font'" class="option-group">
                <label for="font-select">字體</label>
                <select id="font-select" v-model="computedFontFamily">
                  <option value="default">預設字體</option>
                  <option value="cjkFonts">思源字體</option>
                  <option value="jf-openhuninn">粉圓體</option>
                </select>
              </div>

              

              <div v-else-if="activeSetting === 'weight'" class="option-group">
                <label>文字</label>
                <div class="radio-group">
                  <button type="button" class="radio-btn" :class="{ active: exportOptions.weight === 'lighter' }" @click="exportOptions.weight = 'lighter'">
                    <svg width="18" height="18" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                      <text x="6" y="18" font-size="14" font-weight="300" fill="currentColor">A</text>
                    </svg>
                  </button>
                  <button type="button" class="radio-btn" :class="{ active: exportOptions.weight === 'normal' }" @click="exportOptions.weight = 'normal'">
                    <svg width="18" height="18" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                      <text x="6" y="18" font-size="14" font-weight="600" fill="currentColor">A</text>
                    </svg>
                  </button>
                  <button type="button" class="radio-btn" :class="{ active: exportOptions.weight === 'bold' }" @click="exportOptions.weight = 'bold'">
                    <svg width="18" height="18" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                      <text x="6" y="18" font-size="14" font-weight="900" fill="currentColor">A</text>
                    </svg>
                  </button>
                </div>
                <div class="option-group">
                  <label>顏色</label>
                  <ColorPicker :color="exportOptions.color" :alpha="1" @update:color="v => (exportOptions.color = v)" />
                </div>
                <div class="option-group">
                  <label for="font-select">字體</label>
                  <select id="font-select" v-model="computedFontFamily">
                    <option value="default">預設字體</option>
                    <option value="cjkFonts">思源字體</option>
                    <option value="jf-openhuninn">粉圓體</option>
                  </select>
                </div>
              </div>

              <div v-else-if="activeSetting === 'align'" class="option-group">
                <label>對齊</label>
                <div class="radio-group">
                  <button type="button" class="radio-btn" :class="{ active: computedAlignment === 'left' }" @click="computedAlignment = 'left'">
                    <svg width="22" height="18" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M3 6h14" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M3 12h10" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M3 18h16" stroke="currentColor" stroke-width="1.7"/>
                    </svg>
                  </button>
                  <button type="button" class="radio-btn" :class="{ active: computedAlignment === 'center' }" @click="computedAlignment = 'center'">
                    <svg width="22" height="18" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M5 6h14" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M7 12h10" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M3 18h18" stroke="currentColor" stroke-width="1.7"/>
                    </svg>
                  </button>
                  <button type="button" class="radio-btn" :class="{ active: computedAlignment === 'right' }" @click="computedAlignment = 'right'">
                    <svg width="22" height="18" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M7 6h14" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M11 12h10" stroke="currentColor" stroke-width="1.7"/>
                      <path d="M5 18h16" stroke="currentColor" stroke-width="1.7"/>
                    </svg>
                  </button>
                </div>
              </div>


              <template v-else-if="activeSetting === 'stroke'">
                <div class="option-group" :class="{ disabled: !exportOptions.stroke }">
                  <label>描邊顏色</label>
                  <ColorPicker :color="exportOptions.strokeColor" :alpha="1" @update:color="v => (exportOptions.strokeColor = v)" />
                </div>
                <div class="option-group" :class="{ disabled: !exportOptions.stroke }">
                  <label for="stroke-width-select">描邊粗細</label>
                  <select id="stroke-width-select" v-model="exportOptions.strokeWidth">
                    <option :value="1">細</option>
                    <option :value="2">中</option>
                    <option :value="3">粗</option>
                    <option :value="4">很粗</option>
                  </select>
                </div>
              </template>

               <template v-else-if="activeSetting === 'border'">
                 <div class="warning-box">
                   <div class="warning-icon">⚠️</div>
                   <div class="warning-text">
                     <strong>功能故障警告</strong><br>
                     留白功能目前存在技術問題，可能導致輸出圖片異常。建議暫時避免使用此功能。
                   </div>
                 </div>
                 <div class="option-group" :class="{ disabled: !exportOptions.borderEnabled }">
                   <label>背景顏色</label>
                   <ColorPicker 
                     :color="exportOptions.backgroundColor" 
                     :alpha="exportOptions.backgroundOpacity"
                     @update:color="v => (exportOptions.backgroundColor = v)"
                     @update:alpha="v => (exportOptions.backgroundOpacity = v)"
                   />
                 </div>
                 <div class="option-group" :class="{ disabled: !exportOptions.borderEnabled }">
                   <label>邊框顏色</label>
                   <ColorPicker :color="exportOptions.borderColor" :alpha="1" @update:color="v => (exportOptions.borderColor = v)" />
                 </div>
                 <div class="option-group" :class="{ disabled: !exportOptions.borderEnabled }">
                   <label for="padding-range">留白</label>
                   <input id="padding-range" type="range" min="0" max="40" step="1" v-model="exportOptions.padding" />
                 </div>
                 <div class="option-group" :class="{ disabled: !exportOptions.borderEnabled }">
                   <label>圓角大小</label>
                   <div class="radio-group cols-5">
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderRadius === 0 }" @click="exportOptions.borderRadius = 0">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <rect x="6" y="4" width="20" height="12" rx="0" fill="none" stroke="currentColor" stroke-width="1.5"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderRadius === 5 }" @click="exportOptions.borderRadius = 5">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <path d="M6 16 H26 V4 H13 A5 5 0 0 0 8 9 V16 Z" fill="none" stroke="currentColor" stroke-width="1.5"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderRadius === 10 }" @click="exportOptions.borderRadius = 10">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <path d="M6 16 H26 V4 H16 A10 10 0 0 0 6 14 V16 Z" fill="none" stroke="currentColor" stroke-width="1.5"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderRadius === 15 }" @click="exportOptions.borderRadius = 15">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <path d="M6 16 H26 V4 H21 A15 15 0 0 0 6 19 V16 Z" fill="none" stroke="currentColor" stroke-width="1.5"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderRadius === 20 }" @click="exportOptions.borderRadius = 20">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <path d="M6 16 H26 V4 H24 A20 20 0 0 0 6 22 V16 Z" fill="none" stroke="currentColor" stroke-width="1.5"/>
                       </svg>
                     </button>
                   </div>
                 </div>
                 <div class="option-group" :class="{ disabled: !exportOptions.borderEnabled }">
                   <label>邊框粗細</label>
                   <div class="radio-group cols-4">
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderWidth === 1 }" @click="exportOptions.borderWidth = 1">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <rect x="6" y="4" width="20" height="12" rx="4" fill="none" stroke="currentColor" stroke-width="1"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderWidth === 2 }" @click="exportOptions.borderWidth = 2">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <rect x="6" y="4" width="20" height="12" rx="4" fill="none" stroke="currentColor" stroke-width="2"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderWidth === 3 }" @click="exportOptions.borderWidth = 3">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <rect x="6" y="4" width="20" height="12" rx="4" fill="none" stroke="currentColor" stroke-width="3"/>
                       </svg>
                     </button>
                     <button type="button" class="radio-btn" :class="{ active: exportOptions.borderWidth === 4 }" @click="exportOptions.borderWidth = 4">
                       <svg width="32" height="20" viewBox="0 0 32 20" xmlns="http://www.w3.org/2000/svg">
                         <rect x="6" y="4" width="20" height="12" rx="4" fill="none" stroke="currentColor" stroke-width="4"/>
                       </svg>
                     </button>
                   </div>
                 </div>
               </template>

              <template v-else-if="activeSetting === 'shadow'">
                <div class="option-group" :class="{ disabled: !exportOptions.shadow }">
                  <label>陰影顏色</label>
                  <ColorPicker :color="exportOptions.shadowColor" :alpha="1" @update:color="v => (exportOptions.shadowColor = v)" />
                </div>
                <div class="option-group" :class="{ disabled: !exportOptions.shadow }">
                  <label for="shadow-blur">陰影模糊</label>
                  <input id="shadow-blur" type="range" min="0" max="20" step="1" v-model="exportOptions.shadowBlur" />
                </div>
                <div class="option-group" :class="{ disabled: !exportOptions.shadow }">
                  <label for="shadow-offset-x">陰影位移 X</label>
                  <input id="shadow-offset-x" type="range" min="-20" max="20" step="1" v-model="exportOptions.shadowOffsetX" />
                </div>
                <div class="option-group" :class="{ disabled: !exportOptions.shadow }">
                  <label for="shadow-offset-y">陰影位移 Y</label>
                  <input id="shadow-offset-y" type="range" min="-20" max="20" step="1" v-model="exportOptions.shadowOffsetY" />
                </div>
              </template>
            </div>
          </section>

          <aside class="preview-pane">
            <div class="preview">
              <div class="preview-title">預覽</div>
              <div class="preview-box" :style="[previewBoxStyle, { justifyContent: computedAlignment === 'left' ? 'flex-start' : computedAlignment === 'right' ? 'flex-end' : 'center' } ]">
                <div class="preview-text" :style="previewTextStyle">玥餅</div>
              </div>
            </div>
          </aside>
        </div>
      </div>

      <div class="modal-buttons">
        <button @click="confirmExport" class="confirm-btn">下載圖片</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
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
  max-width: 900px;
  width: 92%;
  max-height: 80vh;
  overflow-y: auto;
}

.tabs {
  display: flex;
  gap: .25rem;
  margin-bottom: 1rem;
  border-bottom: 1px solid #e5e7eb; /* 下邊線作為分隔 */
}

.tab {
  padding: .5rem .9rem;
  border: 1px solid transparent;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  background: transparent;
  cursor: pointer;
  color: #555;
  margin-bottom: -1px; /* 讓 active 可蓋住底線 */
}

.tab.active {
  background: #fff;
  color: #333;
  border-color: #e5e7eb;
  border-bottom-color: #fff; /* 蓋住 tabs 的底線 */
  box-shadow: 0 -1px 0 0 #fff inset; /* 補強與內容無縫銜接 */
}

.tab:not(.active):hover {
  background: #f8f8fc;
  color: #333;
}

.tab:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, .3);
  border-color: rgba(102, 126, 234, .4);
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

.option-group.disabled {
  opacity: .5;
  pointer-events: none;
}

.radio-group {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: .5rem;
  width: 100%;
}

.radio-group.cols-4 {
  grid-template-columns: repeat(4, 1fr);
}

.radio-btn {
  padding: .4rem .5rem;
  border: 1px solid #ddd;
  background: #fff;
  border-radius: 6px;
  cursor: pointer;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.radio-btn:hover {
  background: #f8f8fc;
}

.radio-btn.active {
  border-color: #667eea;
  box-shadow: 0 0 0 2px rgba(102,126,234,.15);
  color: #667eea;
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

/* 輸出頁籤佈局 */
.export-layout {
  display: grid;
  grid-template-columns: 160px 1fr 280px;
  gap: 1rem;
  min-height: 260px;
}

.sidebar {
  display: flex;
  flex-direction: column;
  gap: .25rem;
  border-right: 1px solid #e5e7eb;
  padding-right: .5rem;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: .5rem;
  padding: .4rem .5rem;
  border-radius: 6px;
  border: 1px solid transparent;
  background: transparent;
  cursor: pointer;
  color: #333;
}

.sidebar-item:hover {
  background: #f8f8fc;
}

.sidebar-item.active {
  background: #fff;
  border-color: #e5e7eb;
  box-shadow: 0 1px 2px rgba(0,0,0,.04);
}

.sidebar .icon {
  display: inline-flex;
  width: 16px;
  height: 16px;
}

.main {
  display: flex;
  flex-direction: column;
  gap: .75rem;
}

.preview-pane {
  border-left: 1px solid #e5e7eb;
  padding-left: .5rem;
}

.main-header {
  display: flex;
  justify-content: flex-end;
}

.switch-row {
  display: flex;
  align-items: center;
  gap: .5rem;
}

.switch {
  position: relative;
  display: inline-block;
  width: 42px;
  height: 24px;
}

.switch input { display: none; }

.slider {
  position: absolute;
  cursor: pointer;
  top: 0; left: 0; right: 0; bottom: 0;
  background: #e5e7eb;
  border-radius: 9999px;
  transition: background .2s ease;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px; width: 18px;
  left: 3px; top: 3px;
  background: white;
  border-radius: 50%;
  box-shadow: 0 1px 2px rgba(0,0,0,.2);
  transition: transform .2s ease;
}

.switch input:checked + .slider {
  background: #667eea;
}

.switch input:checked + .slider:before {
  transform: translateX(18px);
}

.main-body {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.preview {
  margin-top: .5rem;
}

.preview-title {
  font-weight: bold;
  color: #333;
  margin-bottom: .5rem;
}

.preview-box {
  min-height: 120px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  max-width: 100%;
}

.preview-text {
  font-size: 42px;
}

@media (max-width: 768px) {
  .export-layout {
    grid-template-columns: 1fr;
  }
  .sidebar {
    flex-direction: row;
    flex-wrap: wrap;
    gap: .5rem;
    border-right: 0;
    padding-right: 0;
    margin-bottom: .5rem;
  }
  .sidebar-item {
    flex: 1 1 calc(50% - .5rem);
    justify-content: center;
  }
  .main {
    order: 2;
  }
  .preview-pane {
    order: 3;
    border-left: 0;
    padding-left: 0;
  }
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

.locked-option {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #999;
}

.locked-option svg {
  opacity: 0.6;
}

.sidebar-item.locked {
  opacity: 0.6;
  cursor: not-allowed;
  color: #999;
}

.sidebar-item.locked:hover {
  background: transparent;
}

.locked-section {
  text-align: center;
  padding: 2rem 1rem;
  background: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.locked-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
  color: #6c757d;
  font-weight: 600;
}

.locked-message {
  color: #6c757d;
  margin: 0;
  font-size: 0.9rem;
}

.warning-box {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  padding: 1rem;
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  border-radius: 8px;
  margin-bottom: 1rem;
}

.warning-icon {
  font-size: 1.2rem;
  flex-shrink: 0;
  margin-top: 0.1rem;
}

.warning-text {
  color: #856404;
  font-size: 0.9rem;
  line-height: 1.4;
}

.warning-text strong {
  color: #6c4a00;
}
</style>

