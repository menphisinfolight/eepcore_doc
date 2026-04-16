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

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 12632–12754 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 依開關狀態回傳 `onValue` 或 `offValue` |
| `setValue(value)` | 比對 `onValue` 決定開啟或關閉；checkbox 模式用 `prop('checked')`，button 模式用 `lcs_on()`/`lcs_off()` |
| `readonly(bool)` | checkbox 模式設定 `disabled`；button 模式切換 `lcs_disabled` CSS 類別 |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **雙模式渲染**：`style` 為 `checkbox` 時直接使用原生 checkbox（寬高 25px）；否則使用 `lc_switch` 外掛渲染滑動開關。
- **自動計算寬度**：button 模式下依 `onText`/`offText` 文字長度自動計算開關寬度，英文字元以 `長度/2.7` 折算。
- **label 解綁**：初始化時移除對應 `<label>` 的 `for` 屬性，避免點擊 label 觸發非預期行為。
- **事件觸發**：點擊後延遲 100ms 觸發 `onSelect` 回呼及 `$.triggerValueChanged()`。

## 備註

- 渲染為 `<input type="checkbox">` 加上 `bootstrap-switch` CSS 類別。
