# Place (Editor)

> `EEPRWDTools.Core/Editors/Place.cs` — 22 行
> 繼承：`RWDEditor`

## 用途

**地點選取編輯器**。提供地點搜尋與選取功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **valueType** | PlaceValueType | 列舉 | — | 值類型 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-place` CSS 類別。
