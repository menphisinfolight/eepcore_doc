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

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.promptdialog`（`bootstrap.infolight.js` 第 17819–17934 行）

### 初始化流程

1. 綁定 `.form-ok`、`.form-cancel` 按鈕事件及 `a.close` 關閉事件。
2. 若 `mode` 為 `'panel'`，初始化時隱藏元件（非 Modal 模式）。

### 主要方法

| 方法 | 說明 |
|------|------|
| `show(callback)` | 顯示對話框。`callback` 可為函式（設為 `onOK`）或物件（`{ onOK, onCancel }`，物件模式時隱藏關閉按鈕）。依各欄位的 `readonly` 設定套用唯讀狀態。`mode='panel'` 時用 `show()`，否則用 `modal('show')`。 |
| `getRow()` | 收集所有 `.form-control` 的值，回傳鍵值物件。欄位名稱優先取 `form-field` 的 `field` 屬性。 |
| `ok()` | 收集表單資料後呼叫 `onOK(row)` 回呼，然後關閉 Modal。 |
| `cancel()` | 呼叫 `onCancel()` 回呼（若有），然後關閉 Modal。 |
| `clear()` | 清空所有 `input`、`select` 值；radio/checkbox 取消勾選。 |

### 事件回呼

| 回呼 | 觸發時機 |
|------|----------|
| `onOK(row)` | 按下確定按鈕後，參數為表單收集的資料物件 |
| `onCancel()` | 按下取消按鈕後觸發 |

## 備註

- Render 時內部建立 `PromtForm` 物件，設定 `Mode = FormMode.Dialog`、`FormCls = "bootstrap-promptdialog"`，將 Column 轉換為 `DataForm.Column` 後渲染。
- Column 的 `ToFormColumn()` 方法將 PromptDialog 的欄位定義轉為標準 DataForm 欄位。
- 預設 Editor 為 `Textbox`，可透過設計介面切換為其他編輯器。
