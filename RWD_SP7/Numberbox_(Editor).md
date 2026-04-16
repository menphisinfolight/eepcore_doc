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

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 9440–9502 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 取得原始數值（移除千分位逗號），優先讀取 `srcValue` 屬性 |
| `setValue(value)` | 設定值；非焦點時套用格式化顯示，焦點中直接寫入 |
| `readonly(bool)` | 設定 `disabled` 屬性控制唯讀 |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **焦點切換格式化**：focus 時切為 `type="number"` 並移除逗號顯示原始值；blur 時切回 `type="text"` 並透過 `$.getFormatValue()` 套用 `format` 格式化。
- **srcValue 屬性**：當有設定 `format` 時，原始數值存於 `srcValue` 屬性，顯示值為格式化後文字。
- **禁用滑鼠滾輪**：綁定 `mousewheel` 事件回傳 `false`，防止滾輪意外改值。

## 備註

- 若未設定 `min` 或 `max`，會加上 `no-spiner` CSS 類別以隱藏上下箭頭。
