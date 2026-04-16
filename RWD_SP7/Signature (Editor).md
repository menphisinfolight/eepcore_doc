# Signature (Editor)

> `EEPRWDTools.Core/Editors/Signature.cs` — 33 行
> 繼承：`RWDEditor`

## 用途

**手寫簽名編輯器**。提供手寫簽名功能，支援自訂顏色、背景色及重播功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | 簽名區高度（px） |
| **color** | string | 色彩選擇器 `[ColorEditor]` | — | 筆跡顏色 |
| **background** | string | 色彩選擇器 `[ColorEditor]` | — | 背景顏色 |
| **canReplay** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否可重播簽名過程 |
| **onChange** | string | 事件編輯器 `[ScriptEditor]` | — | 變更事件 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-signature` CSS 類別。
