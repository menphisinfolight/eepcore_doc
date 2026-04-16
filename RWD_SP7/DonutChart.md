# DonutChart

> `EEPRWDTools.Core/Controls/DonutChart.cs` — 66 行
> 繼承：`RWDControl` → `Component`

## 用途

**甜甜圈圖元件**（Donut Chart）。

DonutChart 透過 RemoteName 綁定資料來源，以 LabelField 為類別標籤、ValueField 為數值欄位，繪製中空的環形圖。屬性與 PieChart 完全相同，差異僅在前端渲染樣式。前端渲染為 `<div class="info-donutchart">`。

## JSON 設定範例

```json
{
  "type": "donutchart",
  "id": "dcStatus",
  "remoteName": "qryStatus",
  "labelField": "STATUS",
  "valueField": "COUNT",
  "title": "狀態分佈",
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

- DonutChart 與 PieChart 的程式碼結構完全相同（66 行，屬性一致），唯一差異是 CSS class：`info-donutchart` vs `info-piechart`，前端以不同樣式渲染。
- 適用於需要在圖表中央放置摘要數字或標籤的場景。
- 資料綁定方式同 PieChart，直接使用 LabelField + ValueField，不需 DataFields 集合。
