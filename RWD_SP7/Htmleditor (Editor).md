# Htmleditor (Editor)

> `EEPRWDTools.Core/Editors/Htmleditor.cs` — 31 行
> 繼承：`RWDEditor`

## 用途

**HTML 富文字編輯器**。提供所見即所得的 HTML 編輯功能，支援圖片插入。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 200 | 編輯器高度（px） |
| **imageHeight** | int | 數字框 `[NumberboxEditor]` | 150 | 插入圖片預設高度（px） |
| **imageFolder** | string | 文字 | — | 圖片上傳資料夾 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **htmlAvailable** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否允許直接輸入 HTML |

## 備註

- 渲染為 `<div>` 加上 `bootstrap-htmleditor` CSS 類別。
