# LineChart

> `EEPRWDTools.Core/Controls/LineChart.cs` — 92 行
> 繼承：`RWDControl` → `Component`

## 用途

**折線圖元件**（Line Chart）。

LineChart 透過 RemoteName 綁定資料來源，以 KeyField 為 X 軸分類欄位，DataFields 定義一至多條數據線。支援平滑曲線、動畫、圖例顯示，以及混合折線/長條圖（DataField.Style 可設為 `Line` 或 `Bar`）。前端渲染為 `<div class="info-linechart">`。

## JSON 設定範例

```json
{
  "type": "linechart",
  "id": "lcSales",
  "remoteName": "qrySales",
  "keyField": "MONTH",
  "title": "月銷售趨勢",
  "groupByKey": true,
  "smooth": true,
  "animate": true,
  "legendShow": true,
  "legendLocation": "ne",
  "legendPlacement": "outsideGrid",
  "width": 0,
  "height": 500,
  "dataFields": [
    {
      "label": "營收",
      "field": "REVENUE",
      "showLine": true,
      "lineWidth": 2,
      "markerStyle": "filledCircle",
      "showPointLabels": false,
      "formatString": "$%.2f",
      "style": "Line"
    },
    {
      "label": "成本",
      "field": "COST",
      "showLine": true,
      "lineWidth": 1,
      "markerStyle": "diamond",
      "showPointLabels": true,
      "formatString": "%.2f",
      "style": "Bar"
    }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **keyField** | string | 欄位選擇器 `[ColumnEditor]` | — | X 軸分類欄位 |
| **dataFields** | List\<DataField\> | 集合編輯器 `[CollectionEditor]` | [] | 數據欄位集合（見下方子屬性） |
| **title** | string | 多語系 `[Localization("id")]` | — | 圖表標題 |
| **groupByKey** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否依 KeyField 分群 |
| **smooth** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否平滑曲線 |
| **animate** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否啟用動畫 |
| **legendShow** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示圖例 |
| **legendLocation** | string | 下拉選單 `[ItemsEditor]` | "ne" | 圖例位置（nw/n/ne/e/se/s/sw/w） |
| **legendPlacement** | string | 下拉選單 `[ItemsEditor]` | "insideGrid" | 圖例放置方式（insideGrid/outsideGrid） |
| **width** | int | 數字框 `[NumberboxEditor]` | 0 | 寬度（0 = 自動） |
| **height** | int | 數字框 `[NumberboxEditor]` | 400 | 高度（px） |
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 載入前事件（參數：param） |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊事件（參數：ev, si, pi, data） |
| **queryObj** | string | — | — | 查詢物件 |

### DataField 子屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **label** | string | 多語系 `[Localization("field")]` | — | 數據線標籤 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 對應資料欄位 |
| **showLine** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否顯示線條 |
| **lineWidth** | int | 數字框 `[NumberboxEditor]` | 2 | 線條寬度 |
| **markerStyle** | string | 下拉選單 `[ItemsEditor]` | "filledCircle" | 標記樣式（diamond/circle/square/x/plus/dash/filledDiamond/filledCircle/filledSquare） |
| **showPointLabels** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示資料點標籤 |
| **formatString** | string | 下拉選單（可編輯）`[ItemsEditor]` | — | 格式字串（%d/%.2f/$%.2f 等） |
| **style** | string | 下拉選單（可編輯）`[ItemsEditor]` | — | 顯示樣式（Line/Bar），可實現混合圖 |

## 前端行為（JavaScript）

| 項目 | 說明 |
|------|------|
| **原始檔** | `jquery.infolight.chart.js` |
| **jQuery 外掛** | `$.fn.linechart`（透過 `$.createObj('linechart', ...)` 建立） |
| **CSS 選擇器** | `.info-linechart` |
| **圖表引擎** | jqPlot（`$.jqplot`） |
| **渲染器** | 折線預設使用 jqPlot 內建折線渲染，DataField.style 為 `Bar` 時使用 `$.jqplot.BarRenderer` |

### 方法

| 方法 | 說明 |
|------|------|
| `init` | 讀取 `data-options` 解析設定，設定寬高、呼叫 `load`，若有 `onClick` 則綁定 `jqplotDataClick` 事件 |
| `load` | 組合查詢參數（whereStr、whereItems、pageSize），呼叫 `onBeforeLoad` 回呼後透過 `$.loadData` 向後端取資料，有資料則呼叫 `loadData`，無資料則隱藏元件 |
| `loadData` | 銷毀既有 plot → 若 `groupByKey` 為 true 則呼叫 `$.groupSum` 聚合 → 依 `dataFields` 組裝 series（Line 或 Bar 混合）與 ticks → 呼叫 `$.jqplot` 繪製圖表，X 軸使用 `CategoryAxisRenderer` |
| `setWhere` | 更新 whereStr / whereItems 後重新呼叫 `load`，供 Form 元件篩選條件聯動 |
| `resize` | 呼叫 `plot.replot({ resetAxes: true })` 重繪以適應容器尺寸變化 |

### 與 Form 的整合（bootstrap.infolight.js）

- `form('getLineChart')` 透過 `.info-linechart` 找出 `queryObj` 與 Form id 相符的折線圖元件。
- Form 設定篩選條件時自動呼叫 `linechart('setWhere', whereItems)` 連動更新。
- Tab 切換時自動偵測寬度並呼叫 `resize` 重繪。
- 視窗 resize 事件（200ms debounce）會對所有 `.info-linechart` 呼叫 `resize`。

## 備註

- LineChart 與 BarChart 共用相同的圖表共通屬性模式（RemoteName、KeyField、DataFields、圖例、尺寸、事件），差異在於 LineChart 獨有 `Smooth` 屬性，BarChart 獨有 `Stack` 和 `BarDirection`。
- DataField 的 `Style` 屬性允許在折線圖中混入長條圖系列，實現 combo chart。
- `width` 設為 0 時由容器自動決定寬度。
- `groupByKey` 為 true 時，資料會依 KeyField 分群再繪製。
