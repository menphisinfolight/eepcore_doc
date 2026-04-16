# Numberbox (Editor)

> `EEPRWDTools.Core/Editors/Numberbox.cs` — 58 行
> 繼承：`RWDEditor`

## 用途

**數字輸入框編輯器**。用於數值類欄位，渲染為 `<input type="number">`，支援最大最小值限制、格式化及對齊方式。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **min** | int? | 數字框 `[NumberboxEditor]` | — | 最小值 |
| **max** | int? | 數字框 `[NumberboxEditor]` | — | 最大值 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **textAlign** | string | 下拉選項 `[ItemsEditor]` | "right" | 文字對齊（left / right） |
| **format** | string | 文字 | — | 數字格式化字串 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onBlur** | string | 事件編輯器 `[ScriptEditor]` | — | 失焦事件 |

## 備註

- 若未設定 `min` 或 `max`，會加上 `no-spiner` CSS 類別以隱藏上下箭頭。
