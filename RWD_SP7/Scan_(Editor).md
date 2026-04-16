# Scan (Editor)

> `EEPRWDTools.Core/Editors/Scan.cs` — 23 行
> 繼承：`RWDEditor`

## 用途

**掃描輸入編輯器**。支援條碼/QR Code 掃描輸入，掃描後可透過事件處理掃描結果。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onScan** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 掃描完成事件（預設回傳 `return value;`） |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-scan` CSS 類別。
- `onScan` 事件可對掃描值做前處理後回傳。
