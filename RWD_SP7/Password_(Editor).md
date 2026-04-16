# Password (Editor)

> `EEPRWDTools.Core/Editors/Password.cs` — 20 行
> 繼承：`RWDEditor`

## 用途

**密碼輸入框編輯器**。渲染為 `<input type="password">`，輸入內容以遮罩顯示。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 備註

- 極簡編輯器，僅提供唯讀控制。
