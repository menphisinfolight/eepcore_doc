# PromptDialog

> `EEPRWDTools.Core/Controls/PromptDialog.cs` — 72 行
> 繼承：`RWDControl` → `Component`

## 用途

**提示對話框元件**（Prompt Dialog）。

PromptDialog 提供一個彈出式對話框表單，用於向使用者收集輸入資料。內部建立 `PromtForm`（Dialog 模式的 DataForm），將定義的欄位集合渲染為表單欄位。

## JSON 設定範例

```json
{
  "type": "promptdialog",
  "id": "pdInput",
  "title": "請輸入資料",
  "horizontalColumnsCount": 1,
  "columns": [
    {
      "title": "備註",
      "field": "REMARK",
      "newRow": false,
      "span": 1,
      "editor": { "type": "textbox" }
    }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 `[Localization]` | — | 對話框標題（支援多語系，同時傳入 data-options） |
| **horizontalColumnsCount** | int | 數字框 `[NumberboxEditor]` | — | 水平欄數 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 表單欄位集合 |

### Column 子項目

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **title** | string | 文字 `[Localization]` | — | 欄位標題（支援多語系） |
| **field** | string | 文字 | — | 欄位名稱 |
| **newRow** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否換行 |
| **span** | int | 數字框 `[NumberboxEditor]` | — | 佔用欄數 |
| **editor** | RWDEditor | 編輯器選擇 `[EditorOptionEditor]` | Textbox | 輸入元件類型 |

## 備註

- Render 時內部建立 `PromtForm` 物件，設定 `Mode = FormMode.Dialog`、`FormCls = "bootstrap-promptdialog"`，將 Column 轉換為 `DataForm.Column` 後渲染。
- Column 的 `ToFormColumn()` 方法將 PromptDialog 的欄位定義轉為標準 DataForm 欄位。
- 預設 Editor 為 `Textbox`，可透過設計介面切換為其他編輯器。
