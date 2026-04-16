# SvgChart

> `EEPRWDTools.Core/Controls/SvgChart.cs` — 105 行
> 繼承：`RWDControl` → `Component`

## 用途

**SVG 圖表元件**（SVG Chart）。

SvgChart 載入 SVG 向量圖檔，支援兩種模式：座位圖（seat）和路線圖（route）。座位圖模式可實現座位選取功能；路線圖模式可在 SVG 上繪製動態路線動畫。前端渲染為 `<div class="info-svgchart">`。

## JSON 設定範例

```json
{
  "type": "svgchart",
  "id": "scSeat",
  "remoteName": "qrySeatStatus",
  "svgUrl": "theater.svg",
  "svgLeft": 0,
  "svgTop": 0,
  "svgWidth": 800,
  "svgHeight": 600,
  "svgRate": 2,
  "chartType": "seat",
  "valueField": "STATUS",
  "chartTitle": "座位圖",
  "subTitle": "",
  "roam": false,
  "width": 0,
  "height": 600,
  "separator": ",",
  "selectedColor": "#bf0e08",
  "selectColor": "#007f00"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **svgUrl** | string | 檔案選擇器 `[DataFileEditor("svg")]` | — | SVG 檔案路徑 |
| **svgLeft** | int | 數字框 `[NumberboxEditor]` | 0 | SVG 左偏移 |
| **svgTop** | int | 數字框 `[NumberboxEditor]` | 0 | SVG 上偏移 |
| **svgWidth** | int | 數字框 `[NumberboxEditor]` | 0 | SVG 寬度 |
| **svgHeight** | int | 數字框 `[NumberboxEditor]` | 0 | SVG 高度 |
| **svgRate** | int | 數字框 `[NumberboxEditor]` | 2 | SVG 縮放比率 |
| **chartType** | string | 下拉選單 `[ItemsEditor]` | "seat" | 圖表類型（seat = 座位圖 / route = 路線圖） |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 數值欄位 |
| **chartTitle** | string | — | — | 圖表主標題 |
| **subTitle** | string | — | — | 圖表副標題 |
| **roam** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否允許縮放/拖曳 |
| **width** | int | 數字框 `[NumberboxEditor]` | 0 | 元件寬度（0 = 自動） |
| **height** | int | 數字框 `[NumberboxEditor]` | 600 | 元件高度（px） |

### Route 群組屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **speed** | int | 數字框 `[NumberboxEditor]` | 80 | 動畫速度 |
| **lineWidth** | int | 數字框 `[NumberboxEditor]` | 5 | 路線線條寬度 |
| **lineOpacity** | int | 數字框 `[NumberboxEditor]` | 1 | 路線透明度（0~1） |
| **lineColor** | string | 顏色選擇器 `[ColorEditor]` | "#c46e54" | 路線顏色 |
| **lineStyle** | string | 下拉選單 `[ItemsEditor]` | "dotted" | 線條樣式（dotted/dashed/solid） |
| **symbolColor** | string | 顏色選擇器 `[ColorEditor]` | "#c46e54" | 動畫符號顏色 |

### Seat 群組屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **separator** | string | — | — | 座位名稱分隔符號 |
| **selectedColor** | string | 顏色選擇器 `[ColorEditor]` | "#bf0e08" | 已選取（被佔用）顏色 |
| **selectColor** | string | 顏色選擇器 `[ColorEditor]` | "#007f00" | 選取中顏色 |

### 事件

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | 載入前事件（參數：param） |
| **onLoad** | string | 腳本編輯器 `[ScriptEditor]` | 載入事件（參數：row） |
| **onSelectChanged** | string | 腳本編輯器 `[ScriptEditor]` | 選取變更事件（參數：names） |

## 前端行為（JavaScript）

| 項目 | 說明 |
|------|------|
| **原始檔** | `jquery.infolight.chart.js` |
| **jQuery 外掛** | `$.fn.svgchart`（透過 `$.createObj('svgchart', ...)` 建立） |
| **CSS 選擇器** | `.info-svgchart` |
| **圖表引擎** | ECharts（`echarts.init`、`echarts.registerMap`） |

### 方法

| 方法 | 說明 |
|------|------|
| `init` | 讀取 `data-options` 解析設定，設定寬高後呼叫 `loadSvg` |
| `loadSvg` | 以 `echarts.init` 建立圖表實例（Canvas 渲染器），透過 `$.get` 載入 SVG 檔案，呼叫 `echarts.registerMap(id, { svg })` 註冊為地圖，再呼叫 `load` 載入資料。載入失敗時顯示錯誤訊息 |
| `load` | 組合查詢參數，透過 `$.loadData` 向後端取資料，依序呼叫 `onLoad`、`loadData`、`onLoaded` 回呼 |
| `loadData` | 依 `chartType` 分兩種模式：**seat**（座位圖）和 **route**（路線圖），詳見下方說明 |
| `setWhere` | 更新 whereStr / whereItems 後重新呼叫 `load`（注意：程式碼中使用 `geochart` 取得 options，為共用邏輯） |
| `getChanged` | 回傳座位模式下使用者選取的座位名稱陣列（`selectedNames`） |
| `resize` | 呼叫 ECharts 實例的 `chartObj.resize()` |

### Seat 模式（座位圖）

- 將已佔用座位（資料中 valueField 有值的項目）標記為 `selectedColor`，設為 `silent: true` 不可互動。
- 使用者可點選未佔用座位，觸發 ECharts `geoselectchanged` 事件。
- 若設定 `maxCount`，選取數量超過上限時自動取消最新選取（透過 `geoUnSelect` action）。
- 若設定 `separator`，valueField 值會以分隔符號拆分為多個座位名稱。
- 選取變更時呼叫 `onSelectChanged(selectedNames)` 回呼。
- `svgLeft`/`svgTop` 有值時使用絕對定位；否則使用 `layoutCenter: ['50%', '50%']` 置中，大小為 `svgRate * 100%`。

### Route 模式（路線圖）

- 使用 ECharts `type: 'lines'` + `coordinateSystem: 'geo'` + `polyline: true` 繪製連續路線。
- valueField 值格式為逗號分隔的座標（取前兩個值為 x, y）。
- 內建車輛動畫效果（SVG path 圖示），可設定速度（`speed`）、線條寬度/顏色/透明度/樣式。

### 與 Form / Seat 元件的整合

- SvgChart 不在 `bootstrap.infolight.js` 的 Form `getXxxChart` / `setWhere` 整合中。
- **Seat 元件**（`$.fn.seat`）是獨立的表單元件，開啟 Modal 後內嵌一個 SvgChart（座位模式），使用者選取座位後將結果寫回 input 值。
- 視窗 resize 事件（200ms debounce）會對所有 `.info-svgchart` 呼叫 `resize`。

## 備註

- SvgChart 與 GeoChart 同屬「向量圖表」類型，共用 Roam、ChartTitle、SubTitle 等屬性，但 SvgChart 載入 SVG 檔而非 GeoJSON。
- 座位圖模式（seat）：SVG 中的各元素代表座位，依資料著色，使用者可點選座位，觸發 onSelectChanged 事件回傳選取的座位名稱。
- 路線圖模式（route）：在 SVG 地圖上繪製動態路線動畫，可設定速度、線條樣式和顏色。
- `svgRate` 控制 SVG 的縮放比例，預設為 2。
- 注意原始碼中 `lineColor` 屬性名稱為小寫開頭（`lineColor`），與其他屬性的 Pascal 命名風格不一致。
