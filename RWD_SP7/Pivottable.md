# Pivottable

> `EEPRWDTools.Core/Controls/Pivottable.cs` — 82 行
> 繼承：`RWDControl` → `Component`

## 用途

**樞紐分析表元件**（Pivot Table）。

Pivottable 提供互動式樞紐分析功能，可從資料來源載入資料後，由使用者拖放欄位進行多維度交叉分析。支援自訂聚合函數、渲染器，以及預設的列/欄配置。

## JSON 設定範例

```json
{
  "type": "pivottable",
  "id": "pvSales",
  "remoteName": "cmdSalesData",
  "panelHeight": 200,
  "digitsAfterDecimal": 2,
  "defaultColumn": "AMOUNT",
  "columns": [
    { "title": "產品", "field": "PRODUCT", "tableName": "" }
  ],
  "rows": [
    { "title": "區域", "field": "REGION", "tableName": "" }
  ],
  "showColumns": [
    { "title": "金額", "field": "AMOUNT", "tableName": "", "hideInValue": false }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源命令 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 欄（Column）區域的欄位集合 |
| **rows** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 列（Row）區域的欄位集合 |
| **showColumns** | List\<ShowColumn\> | 集合編輯器 `[CollectionEditor]` | — | 顯示欄位集合 |
| **defaultColumn** | string | 欄位選擇器 `[ColumnEditor]` | — | 預設值欄位 |
| **whereStr** | string | — | — | 查詢條件字串 |
| **panelHeight** | int | 數字框 `[NumberboxEditor]` | 200 | 面板高度（px） |
| **digitsAfterDecimal** | int | 數字框 `[NumberboxEditor]` | 2 | 小數位數 |
| **aggregators** | string | 樞紐聚合器 `[PivotAggregatorsEditor]` | — | 聚合函數設定 |
| **renderers** | string | 樞紐渲染器 `[PivotRendersEditor]` | — | 渲染器設定 |
| **queryObj** | string | — | — | 查詢物件 |
| **onLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 資料載入後事件，簽名 `function(data){}` |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 儲存格點擊事件，簽名 `function(item){}` |

### Column 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題 |
| **field** | string | 欄位名稱 |
| **tableName** | string | 資料表名稱 |

### ShowColumn 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題 |
| **field** | string | 欄位名稱 |
| **tableName** | string | 資料表名稱 |
| **hideInValue** | bool | 是否在值區域中隱藏 |

## 備註

- Render 輸出 `<div class="bootstrap-pivottable">`，前端使用 PivotTable.js 套件。
- `columns` 和 `rows` 使用相同的 `Column` 子類別定義。
- `aggregators` 和 `renderers` 使用專屬的設計介面編輯器（`PivotAggregatorsEditor` / `PivotRendersEditor`）。
