# ReportViewer

> `EEPRWDTools.Core/Controls/ReportViewer.cs` — 90 行
> 繼承：`RWDControl` → `Component`

## 用途

**報表檢視器元件**（Report Viewer）。

ReportViewer 用於在前端嵌入報表，支援查詢面板（QueryForm）、報表參數（Parameter）、及多資料來源（DataSource）。內部會自動建立 QueryForm 提供查詢條件輸入，並將結果傳遞給報表引擎。

## JSON 設定範例

```json
{
  "type": "reportviewer",
  "id": "rptSales",
  "remoteName": "cmdSalesReport",
  "reportName": "SalesReport",
  "visible": true,
  "preload": false,
  "height": 500,
  "fit": true,
  "queryColumns": [
    { "field": "SALES_DATE", "editor": { "type": "datebox" } }
  ],
  "parameters": [
    { "parameterName": "StartDate", "value": "" }
  ],
  "dataSources": [
    { "dataSourceName": "dsDetail", "remoteName": "cmdDetail" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 主要資料來源 RemoteName |
| **reportName** | string | 選單選擇器 `[MenuEditor]`（report） | — | 報表名稱 |
| **queryColumns** | List\<QueryColumn\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 查詢欄位定義 |
| **parameters** | List\<Parameter\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 報表參數 |
| **dataSources** | List\<DataSource\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 額外資料來源 |
| **visible** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否預設可見 |
| **preload** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否預先載入報表 |
| **width** | int? | 數字框 `[NumberboxEditor]` | — | 元件寬度（px） |
| **height** | int | 數字框 `[NumberboxEditor]` | 300 | 元件高度（px） |
| **fit** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否自動填滿容器 |

### Parameter 子類別

| 屬性 | 類型 | 說明 |
|------|------|------|
| **parameterName** | string | 報表參數名稱 |
| **value** | string | 參數值 |

### DataSource 子類別

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **dataSourceName** | string | 文字 | 資料來源名稱 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | 資料來源的 RemoteName |

## 備註

- 渲染時輸出 `<div class="bootstrap-reportviewer">`，內含 QueryForm 查詢面板和 `<div class="report">` 報表區域。
- `queryColumns` 標記為 `[DataOption(false)]`，不輸出至前端 data-options，僅用於伺服端建立 QueryForm。
- QueryForm 固定使用 `HorizontalColumnsCount = 2` 和 `QueryMode.Panel`。
