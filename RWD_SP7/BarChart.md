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

## 備註

- BarChart 與 LineChart 共用大部分共通屬性（RemoteName、KeyField、DataFields、圖例、尺寸、事件），差異在於 BarChart 獨有 `Stack`（堆疊）和 `BarDirection`（方向），而 LineChart 獨有 `Smooth`。
- BarChart 的 DataField 較 LineChart 簡化，沒有 ShowLine、LineWidth、MarkerStyle、Style 等線條相關屬性。
- `barDirection` 設為 `horizontal` 可將長條圖轉為水平方向顯示。
- `stack` 為 true 時，多組系列會堆疊在同一長條上，適合顯示組成比例。
