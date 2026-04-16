# DataPanel

> `EEPRWDTools.Core/Controls/DataPanel.cs` — 60 行
> 繼承：`RWDControl` → `Component`

## 用途

**資料面板元件**（Data Panel）。

DataPanel 用來在頁面上呈現一組表單欄位，可選擇性地以 `<fieldset>` + `<legend>` 包裹並顯示標題。內部委派 `DataForm` 進行實際的表單欄位渲染，支援多欄水平排列。通常用於將表單欄位分區顯示（例如「基本資料」、「聯絡資訊」等區塊）。

## JSON 設定範例

```json
{
  "type": "datapanel",
  "id": "panelBasic",
  "remoteName": "cmdEmployee",
  "title": "基本資料",
  "horizontalColumnsCount": 2,
  "columns": [
    { "field": "EMP_NAME", "editor": { "type": "textbox" } },
    { "field": "EMP_NO", "editor": { "type": "textbox" } }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源 RemoteName |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 表單欄位定義（使用 DataForm.Column） |
| **horizontalColumnsCount** | int | 數字框 `[NumberboxEditor]`（min=1, 預設=1） | 1 | 水平欄位數（每列顯示幾個欄位） |
| **title** | string | 文字（多語系） `[Localization]` | — | 面板標題（顯示於 fieldset legend） |

## 備註

- 當 `Title` 有值時，會以 `<fieldset><legend>` 包裹整個表單區塊；無值時直接渲染表單。
- 內部建立 `DataForm` 物件來執行渲染，將 Columns 和 HorizontalColumnsCount 傳入。
- `PId` 為內部屬性（`DataOption(false)`），不輸出至前端 data-options。
