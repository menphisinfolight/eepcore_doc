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

## 備註

- PieChart 與 DonutChart 的屬性結構完全相同，差異僅在前端渲染：PieChart 為實心圓餅，DonutChart 中間挖空為甜甜圈。
- 與 LineChart/BarChart 不同，PieChart 不使用 DataFields 集合，而是直接指定單一 LabelField 和 ValueField。
- `sliceMargin` 可設定切片間的間距，增加視覺區隔效果。
- `dataLabelStyle` 控制顯示內容：`label` 顯示類別名稱、`value` 顯示數值、`percent` 顯示百分比。
