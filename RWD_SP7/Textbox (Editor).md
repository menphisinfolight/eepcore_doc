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

## 備註

- 若設定 `iconCls`，會額外包一層 `div.input-group` 並在右側加上圖示按鈕。
- 支援 FontAwesome（`fa` 開頭）和 Glyphicon 兩種圖示。
- 渲染為 `<input type="text">` 標籤。
