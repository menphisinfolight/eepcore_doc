# Switch (Editor)

> `EEPRWDTools.Core/Editors/Switch.cs` — 36 行
> 繼承：`RWDEditor`

## 用途

**開關切換編輯器**。提供 On/Off 二值切換，可自訂顯示文字和對應值。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **onText** | string | 文字 | "Yes" | 開啟時顯示文字 |
| **onValue** | string | 文字 | "true" | 開啟時的值 |
| **offText** | string | 文字 | "No" | 關閉時顯示文字 |
| **offValue** | string | 文字 | "false" | 關閉時的值 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **style** | string | 下拉選項 `[ItemsEditor]` | "button" | 樣式（button / checkbox） |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 切換事件 |

## 備註

- 渲染為 `<input type="checkbox">` 加上 `bootstrap-switch` CSS 類別。
