# Creator (Editor)

> `EEPRWDTools.Core/Editors/Creator.cs` — 22 行
> 繼承：`RWDEditor`

## 用途

**建立者欄位編輯器**。自動記錄資料建立者資訊，欄位預設為唯讀。可搭配日期欄位記錄建立時間。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **dateField** | string | 欄位選擇器 `[ColumnEditor("parent")]` | — | 對應的建立日期欄位 |

## 備註

- Render 時自動加入 `readonly: true` 到 data-options。
- 渲染為 `<input>` 加上 `bootstrap-creator` CSS 類別。
