# Textbox (Editor)

> `EEPRWDTools.Core/Editors/Textbox.cs` — 75 行
> 繼承：`RWDEditor`

## 用途

**文字輸入框編輯器**。最基本的欄位編輯器，用於一般文字輸入。支援最大長度限制、對齊方式、提示文字、圖示按鈕等功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **maxLength** | int? | 數字框 `[NumberboxEditor]` | — | 最大輸入長度 |
| **textAlign** | TextAlign | 列舉 | — | 文字對齊方式 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **iconCls** | string | 圖示選擇器 `[IconRWDEditor]` | — | 右側圖示按鈕（支援 glyphicon / FontAwesome） |
| **onBlur** | string | 事件編輯器 `[ScriptEditor]` | — | 失焦事件 |

## 前端行為（JavaScript）

Textbox 沒有獨立的 `$.createObj` 元件，而是透過 **datagrid / tablegrid 的 `editors.textbox`** 定義來初始化（`$.fn.datagrid.defaults.editors.textbox`）。

### 初始化（init）

在 DataGrid 或 DataForm 的編輯欄位容器中，建立 `<input type="text" class="form-control"/>`，並依據 options 設定：

| 選項 | 前端處理 |
|------|----------|
| `maxLength` | 設定 `maxlength` 屬性 |
| `onBlur` | 綁定 `blur` 事件回呼 |
| `textAlign` | 透過 `css('text-align')` 設定對齊 |
| `readonly` | 設定 `disabled` 屬性 |
| `prompt` | 設定 `placeholder` 屬性 |

### getValue / setValue

- `getValue`：直接回傳 `$(target).val()`。
- `setValue`：直接設定 `$(target).val(value)`。

### readonly

透過 `$(target).attr('disabled', value)` 切換唯讀狀態。

> **註：** Textbox 為最基本的編輯器，所有邏輯皆在 datagrid editors 定義中完成，無額外 jQuery plugin。

## 備註

- 若設定 `iconCls`，會額外包一層 `div.input-group` 並在右側加上圖示按鈕。
- 支援 FontAwesome（`fa` 開頭）和 Glyphicon 兩種圖示。
- 渲染為 `<input type="text">` 標籤。
