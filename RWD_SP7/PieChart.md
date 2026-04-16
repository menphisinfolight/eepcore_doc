# PieChart

> `EEPRWDTools.Core/Controls/PieChart.cs` — 66 行
> 繼承：`RWDControl` → `Component`

## 用途

**圓餅圖元件**（Pie Chart）。

PieChart 透過 RemoteName 綁定資料來源，以 LabelField 為類別標籤、ValueField 為數值欄位，繪製圓餅圖。支援資料標籤樣式切換（label/value/percent）、切片間距、圖例。前端渲染為 `<div class="info-piechart">`。

## JSON 設定範例

```json
{
  "type": "piechart",
  "id": "pcMarket",
  "remoteName": "qryMarketShare",
  "labelField": "REGION",
  "valueField": "AMOUNT",
  "title": "市場佔比",
  "showDataLabels": true,
  "dataLabelStyle": "percent",
  "sliceMargin": 2,
  "legendShow": true,
  "legendLocation": "e",
  "legendPlacement": "outsideGrid",
  "legendRows": 0,
  "width": 400,
  "height": 400
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **labelField** | string | 欄位選擇器 `[ColumnEditor]` | — | 類別標籤欄位 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 數值欄位 |
| **title** | string | 多語系 `[Localization("id")]` | — | 圖表標題 |
| **showDataLabels** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否顯示資料標籤 |
| **dataLabelStyle** | string | 下拉選單 `[ItemsEditor]` | "percent" | 標籤樣式（label/value/percent） |
| **sliceMargin** | int | 數字框 `[NumberboxEditor]` | 0 | 切片間距（px） |
| **legendShow** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示圖例 |
| **legendLocation** | string | 下拉選單 `[ItemsEditor]` | "ne" | 圖例位置（nw/n/ne/e/se/s/sw/w） |
| **legendPlacement** | string | 下拉選單 `[ItemsEditor]` | "insideGrid" | 圖例放置方式（insideGrid/outsideGrid） |
| **legendRows** | int | 數字框 `[NumberboxEditor]` | 0 | 圖例列數（0 = 自動） |
| **width** | int | 數字框 `[NumberboxEditor]` | 400 | 寬度（px） |
| **height** | int | 數字框 `[NumberboxEditor]` | 400 | 高度（px） |
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 載入前事件（參數：param） |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊事件（參數：ev, si, pi, data） |
| **queryObj** | string | — | — | 查詢物件 |

## 前端行為（JavaScript）

| 項目 | 說明 |
|------|------|
| **原始檔** | `jquery.infolight.chart.js` |
| **jQuery 外掛** | `$.fn.piechart`（透過 `$.createObj('piechart', ...)` 建立） |
| **CSS 選擇器** | `.info-piechart` |
| **圖表引擎** | jqPlot（`$.jqplot`） |
| **渲染器** | `$.jqplot.PieRenderer` |

### 方法

| 方法 | 說明 |
|------|------|
| `init` | 讀取 `data-options` 解析設定，設定寬高、呼叫 `load`，若有 `onClick` 則綁定 `jqplotDataClick` 事件 |
| `load` | 組合查詢參數（whereStr、whereItems、pageSize），呼叫 `onBeforeLoad` 回呼後透過 `$.loadData` 向後端取資料，有資料則呼叫 `loadData`，無資料則隱藏元件 |
| `loadData` | 銷毀既有 plot → 若 `groupByKey` 為 true 則依 `labelField` 聚合 → 將資料轉為 `[label, value]` 陣列 → 呼叫 `$.jqplot` 以 `PieRenderer` 繪製，設定 sliceMargin、showDataLabels、dataLabels 樣式 |
| `setWhere` | 更新 whereStr / whereItems 後重新呼叫 `load`，供 Form 元件篩選條件聯動 |
| `resize` | 呼叫 `plot.replot({ resetAxes: true })` 重繪以適應容器尺寸變化 |

### 與 Form 的整合（bootstrap.infolight.js）

- `form('getPieChart')` 透過 `.info-piechart` 找出 `queryObj` 與 Form id 相符的圓餅圖元件。
- Form 設定篩選條件時自動呼叫 `piechart('setWhere', whereItems)` 連動更新。
- Tab 切換時自動偵測寬度並呼叫 `resize` 重繪。
- 視窗 resize 事件（200ms debounce）會對所有 `.info-piechart` 呼叫 `resize`。

### 資料格式

與 LineChart/BarChart 不同，PieChart 不使用多系列 dataFields，而是將每筆資料轉為 `[labelField, valueField]` 的二元陣列，傳給 jqPlot 的 PieRenderer。動畫固定啟用（`animate: true`）。

## 備註

- PieChart 與 DonutChart 的屬性結構完全相同，差異僅在前端渲染：PieChart 為實心圓餅，DonutChart 中間挖空為甜甜圈。
- 與 LineChart/BarChart 不同，PieChart 不使用 DataFields 集合，而是直接指定單一 LabelField 和 ValueField。
- `sliceMargin` 可設定切片間的間距，增加視覺區隔效果。
- `dataLabelStyle` 控制顯示內容：`label` 顯示類別名稱、`value` 顯示數值、`percent` 顯示百分比。
