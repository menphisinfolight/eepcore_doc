# Textarea (Editor)

> `EEPRWDTools.Core/Editors/Textarea.cs` — 36 行
> 繼承：`RWDEditor`

## 用途

**多行文字輸入框編輯器**。用於需要輸入較長文字的欄位，渲染為 `<textarea>` 標籤。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **maxLength** | int? | 數字框 `[NumberboxEditor]` | — | 最大輸入長度 |
| **rows** | int | 數字框 `[NumberboxEditor]` | 3 | 顯示行數 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 備註

- 當 `maxLength` 有值且大於 0 時，會在 HTML 上設定 `maxlength` 屬性。
