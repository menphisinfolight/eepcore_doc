# Card

> `EEPRWDTools.Core/Controls/Card.cs` — 55 行
> 繼承：`RWDContainerControl` → `RWDControl` → `Component`

## 用途

**卡片容器元件**（Card）。

Card 是一個可摺疊的容器面板，用於將子元件（childrens）包裝在卡片式 UI 中。支援標題、圖示、配色主題、顯示/隱藏控制，以及展開事件。當指定 `url` 時，子元件才會被渲染（延遲載入）。

## JSON 設定範例

```json
{
  "type": "card",
  "id": "cardOrder",
  "title": "訂單資訊",
  "iconCls": "glyphicon glyphicon-list-alt",
  "class": "blue",
  "commands": "moveTop,hide",
  "visible": true,
  "large": false,
  "horizontalColumnsCount": 200,
  "url": "/api/module",
  "childrens": []
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 `[Localization]` | — | 卡片標題（支援多語系） |
| **iconCls** | string | 圖示選擇器 `[IconRWDEditor("bootstrap")]` | — | Bootstrap 圖示 CSS 類別 |
| **class** | string | 下拉選擇 `[ItemsEditor]` | — | 配色主題（dark red / dark orange / dark blue / dark green / dark purple / dark yellow / dark gray / red / orange / blue / green / yellow / purple / gray），不傳入 data-options |
| **commands** | string | 多選 `[ItemsEditor]` | — | 卡片指令（moveTop / moveBottom / hide / open） |
| **url** | string | 文字 | — | 延遲載入 URL，有值時才渲染子元件 |
| **visible** | bool | 勾選 `[CheckboxEditor]` | `true` | 是否預設顯示 |
| **large** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否使用大尺寸模式 |
| **horizontalColumnsCount** | int | 數字框 `[NumberboxEditor]` | 200 | 水平欄數（不傳入 data-options） |
| **onOpen** | string | 腳本編輯器 `[ScriptEditor]` | — | 卡片展開時觸發，簽名 `function(){ return true; }` |

## 前端行為（JavaScript）

Card 元件擁有獨立的前端 jQuery plugin，定義於 `/js/jquery-card/jquery.card.js`，透過 `$.createObj('card', ...)` 建立。

### 初始化與布局

**`$.fn.card = $.createObj('card', { ... })`**（`jquery.card.js` 約第 171 行）：

| 預設屬性 | 值 | 說明 |
|----------|-----|------|
| `width` | 320 | 單一卡片寬度（px） |
| `margin` | 15 | 卡片間距（px） |
| `toolbar` | `'open'` | 是否顯示工具列 |
| `animateSpeed` | 400 | 動畫時間（ms） |
| `dragAction` | `'auto'` | 拖曳行為 |

**`init(jq, options)`**：
- 合併 `data-options` 與傳入的 options。
- 根據 `large` 屬性計算 `span`（1 或 2），寬度 = `(defaults.width + margin) * span - margin`。
- 初始化時所有卡片設為 `width: 0, height: 0` 並加上 `card-hidden` CSS 類別。
- 呼叫 `createLayout` 建立 header 結構。

**`createLayout(jq)`**：
- 建立 `.card-header`（含圖示與標題）。
- 根據 `commands` 建立 popover 指令按鈕（moveTop / moveBottom / hide / open）。
- `.card-body` 高度 = `opts.height - 2 - header.outerHeight()`。

### 卡片排版（Masonry 式瀑布流）

**`relocation(jq, callback)`**（約第 324 行）：
- 將所有卡片依 `index` 排序後逐一計算位置。
- 從左至右、從上至下排列，尋找各欄最低底部位置放置新卡片（瀑布流演算法）。
- 計算完成後執行 `animate` 或指定的 callback。

**`load(jq)`**（約第 378 行）：
- 依序對非隱藏的卡片執行展開動畫（從中心點放大），動畫結束後呼叫 `loadUrl` 和 `loadChildren`。

### 延遲載入

**`loadUrl(jq)`**（約第 405 行）：
- 若 `opts.url` 有值，在 `.card-body` 中插入 `<iframe>` 載入指定 URL。

**`loadChildren(jq)`**（約第 417 行）：
- 觸發卡片內的 chart 元件（linechart / barchart / piechart）執行 `resize`，解決隱藏狀態下 jqPlot 無法正確初始化的問題。

### 指令方法

| 方法 | 行為 |
|------|------|
| `open` | 觸發 `onOpen` 事件回呼；若有 `url`，在主框架新增 tab 或開新視窗 |
| `hide` | 設定 `card-hidden` 屬性，執行縮小動畫後觸發所有卡片重新排版 |
| `show` | 移除 `card-hidden`，重新排版後執行放大動畫，完成後載入 URL 和子元件 |
| `moveTop` | 將卡片 index 改為前一張的 index，前方所有卡片 index +1，重新排版 |
| `moveBottom` | 將卡片 index 改為最後一張的 index，後方所有卡片 index -1，重新排版 |

### 狀態持久化

卡片的排列順序和顯示/隱藏狀態透過 `$.fn.card.save()` 儲存，`hide`、`show`、`moveTop`、`moveBottom` 操作後均自動呼叫。

### systemInfoCard 整合

`jquery.infolight.systeminfocard.js` 負責首頁卡片的動態建立：
- 從 Server 取得 `SYS_CARDS` 資料（`runtimeSYS_CARDS`），根據 `CARDTYPE`（url / page / chart / slide / calendar / excel / menulist / gauge / todo / history / notify）建立不同類型的卡片內容。
- 建立 `<div class="info-card {color}">` 元素後呼叫 `$.fn.card.load()` 啟動排版與動畫。
- 擴充了 `reloadChart` 方法，用於重新載入卡片內的圖表。

## 備註

- Card 繼承 `RWDContainerControl`，支援 `childrens` 子元件集合。
- Render 時產生 `<div class="info-card card-hidden {class}">` 標籤，CSS class 包含 `card-hidden` 表示初始隱藏，由前端 JS 控制顯示。
- `class` 和 `horizontalColumnsCount` 標記 `[DataOption(false)]`，不會輸出到 HTML `data-options` 屬性中。
