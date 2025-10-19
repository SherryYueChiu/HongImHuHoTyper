# 台語方音符號輸入法 (HongImHuHoTyper)

一個對手機和電腦都友善的台語方音符號輸入法應用程式。

## 功能特色

- 🎯 **響應式設計**：完美支援手機和電腦使用
- ⌨️ **虛擬鍵盤**：觸控友善的台語方音符號鍵盤
- 📝 **雙重輸入**：支援虛擬鍵盤和文字輸入欄
- 🎨 **視覺化展示**：橫書句子配合縱向方音符號顯示
- 🎵 **音調標註**：自動在最後一個方音符號右上角標註音調
- 📷 **圖片輸出**：一鍵輸出透明背景圖片
- 🎨 **現代化UI**：美觀的漸層背景和毛玻璃效果

## 技術架構

- **前端框架**：Vue 3 + TypeScript
- **建構工具**：Vite
- **樣式設計**：CSS3 + 響應式設計
- **字體支援**：Microsoft JhengHei, PingFang TC

## 專案結構

```
src/
├── components/          # Vue 組件
│   ├── VirtualKeyboard.vue    # 虛擬鍵盤組件
│   ├── ResultDisplay.vue     # 結果展示組件
│   └── InputBar.vue          # 輸入欄組件
├── utils/               # 工具函數
│   └── phoneticConverter.ts  # 台語方音符號轉換工具
├── App.vue              # 主應用組件
└── main.ts              # 應用入口
```

## 快速開始

### 安裝依賴

```bash
npm install
```

### 開發模式

```bash
npm run dev
```

### 建構生產版本

```bash
npm run build
```

## 使用說明

1. **輸入文字**：在上方輸入欄輸入漢字或英文
2. **虛擬鍵盤**：點擊下方虛擬鍵盤輸入台語方音符號
3. **查看結果**：上半部會顯示橫書句子和對應的縱向方音符號
4. **音調標註**：最後一個方音符號會自動標註音調
5. **輸出圖片**：點擊「輸出圖片」按鈕下載透明背景圖片

## 台語方音符號支援

本應用程式支援完整的台語方音符號系統，包括：

- **聲母**：ㄅㄆㄇㄈㄉㄊㄋㄌㄍㄎㄏㄐㄑㄒㄓㄔㄕㄖㄗㄘㄙ
- **韻母**：ㄚㄛㄜㄝㄞㄟㄠㄡㄢㄣㄤㄥㄦㄧㄨㄩ
- **音調**：ˊˇˋ˙ˉ（陰平、陽平、上聲、去聲、入聲）

## 響應式設計

- **桌面版**：完整功能展示，大螢幕優化
- **手機版**：觸控友善，按鍵大小適中
- **平板版**：中等螢幕適配，平衡體驗

## 瀏覽器支援

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## 授權

MIT License

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd) 
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Type Support for `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) to make the TypeScript language service aware of `.vue` types.

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```
