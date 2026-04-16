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

## 備註

- SvgChart 與 GeoChart 同屬「向量圖表」類型，共用 Roam、ChartTitle、SubTitle 等屬性，但 SvgChart 載入 SVG 檔而非 GeoJSON。
- 座位圖模式（seat）：SVG 中的各元素代表座位，依資料著色，使用者可點選座位，觸發 onSelectChanged 事件回傳選取的座位名稱。
- 路線圖模式（route）：在 SVG 地圖上繪製動態路線動畫，可設定速度、線條樣式和顏色。
- `svgRate` 控制 SVG 的縮放比例，預設為 2。
- 注意原始碼中 `lineColor` 屬性名稱為小寫開頭（`lineColor`），與其他屬性的 Pascal 命名風格不一致。
