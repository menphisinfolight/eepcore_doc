# BarChart

> `EEPRWDTools.Core/Controls/BarChart.cs` — 82 行
> 繼承：`RWDControl` → `Component`

## 用途

**長條圖元件**（Bar Chart）。

BarChart 透過 RemoteName 綁定資料來源，以 KeyField 為分類軸，DataFields 定義一至多組數據系列。支援堆疊（Stack）、水平/垂直方向、動畫、圖例顯示。前端渲染為 `<div class="info-barchart">`。

## JSON 設定範例

```json
{
  "type": "barchart",
  "id": "bcCategory",
  "remoteName": "qryCategory",
  "keyField": "CATEGORY",
  "title": "各類別銷量",
  "groupByKey": false,
  "stack": true,
  "animate": true,
  "legendShow": true,
  "legendLocation": "ne",
  "legendPlacement": "insideGrid",
  "barDirection": "vertical",
  "width": 0,
  "height": 400,
  "dataFields": [
    {
      "label": "Q1",
      "field": "Q1_SALES",
      "showPointLabels": true,
      "formatString": "%d"
    },
    {
      "label": "Q2",
      "field": "Q2_SALES",
      "showPointLabels": false
    }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **keyField** | string | 欄位選擇器 `[ColumnEditor]` | — | 分類軸欄位 |
| **dataFields** | List\<DataField\> | 集合編輯器 `[CollectionEditor]` | [] | 數據欄位集合（見下方子屬性） |
| **title** | string | 多語系 `[Localization("id")]` | — | 圖表標題 |
| **groupByKey** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否依 KeyField 分群 |
| **stack** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否堆疊顯示 |
| **animate** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否啟用動畫 |
| **legendShow** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示圖例 |
| **legendLocation** | string | 下拉選單 `[ItemsEditor]` | "ne" | 圖例位置（nw/n/ne/e/se/s/sw/w） |
| **legendPlacement** | string | 下拉選單 `[ItemsEditor]` | "insideGrid" | 圖例放置方式（insideGrid/outsideGrid） |
| **barDirection** | string | 下拉選單 `[ItemsEditor]` | "vertical" | 長條方向（vertical/horizontal） |
| **width** | int | 數字框 `[NumberboxEditor]` | 0 | 寬度（0 = 自動） |
| **height** | int | 數字框 `[NumberboxEditor]` | 400 | 高度（px） |
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 載入前事件（參數：param） |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊事件（參數：ev, si, pi, data） |
| **queryObj** | string | — | — | 查詢物件 |

### DataField 子屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **label** | string | 多語系 `[Localization("field")]` | — | 數據系列標籤 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 對應資料欄位 |
| **showPointLabels** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示資料點標籤 |
| **formatString** | string | 下拉選單（可編輯）`[ItemsEditor]` | — | 格式字串（%d/%.2f/$%.2f 等） |

## 前端行為（JavaScript）

| 項目 | 說明 |
|------|------|
| **原始檔** | `jquery.infolight.chart.js` |
| **jQuery 外掛** | `$.fn.barchart`（透過 `$.createObj('barchart', ...)` 建立） |
| **CSS 選擇器** | `.info-barchart` |
| **圖表引擎** | jqPlot（`$.jqplot`） |
| **渲染器** | `$.jqplot.BarRenderer` |

### 方法

| 方法 | 說明 |
|------|------|
| `init` | 讀取 `data-options` 解析設定，設定寬高、呼叫 `load`，若有 `onClick` 則綁定 `jqplotDataClick` 事件 |
| `load` | 組合查詢參數（whereStr、whereItems、pageSize），呼叫 `onBeforeLoad` 回呼後透過 `$.loadData` 向後端取資料，有資料則呼叫 `loadData`，無資料則隱藏元件 |
| `loadData` | 銷毀既有 plot → 若 `groupByKey` 為 true 則呼叫 `$.groupSum` 聚合 → 依 `barDirection` 決定 X/Y 軸設定（horizontal 時軸互換）→ 呼叫 `$.jqplot` 繪製，支援 `stackSeries`（堆疊）、`barWidth`、`barDirection` |
| `setWhere` | 更新 whereStr / whereItems 後重新呼叫 `load`，供 Form 元件篩選條件聯動 |
| `resize` | 呼叫 `plot.replot({ resetAxes: true })` 重繪以適應容器尺寸變化 |

### 與 Form 的整合（bootstrap.infolight.js）

- `form('getBarChart')` 透過 `.info-barchart` 找出 `queryObj` 與 Form id 相符的長條圖元件。
- Form 設定篩選條件時自動呼叫 `barchart('setWhere', whereItems)` 連動更新。
- Tab 切換時自動偵測寬度並呼叫 `resize` 重繪。
- 視窗 resize 事件（200ms debounce）會對所有 `.info-barchart` 呼叫 `resize`。

### BarDirection 軸交換邏輯

`barDirection` 為 `horizontal` 時，前端會將 X 軸與 Y 軸的設定互換：分類軸（CategoryAxisRenderer + ticks）移至 Y 軸，數值軸移至 X 軸，實現水平長條圖。

## 備註

- BarChart 與 LineChart 共用大部分共通屬性（RemoteName、KeyField、DataFields、圖例、尺寸、事件），差異在於 BarChart 獨有 `Stack`（堆疊）和 `BarDirection`（方向），而 LineChart 獨有 `Smooth`。
- BarChart 的 DataField 較 LineChart 簡化，沒有 ShowLine、LineWidth、MarkerStyle、Style 等線條相關屬性。
- `barDirection` 設為 `horizontal` 可將長條圖轉為水平方向顯示。
- `stack` 為 true 時，多組系列會堆疊在同一長條上，適合顯示組成比例。
