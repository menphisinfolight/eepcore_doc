# Updater (Editor)

> `EEPRWDTools.Core/Editors/Updater.cs` — 20 行
> 繼承：`RWDEditor`

## 用途

**更新者欄位編輯器**。自動記錄資料最後更新者資訊。可搭配日期欄位記錄更新時間。與 Creator 對應。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **dateField** | string | 欄位選擇器 `[ColumnEditor("parent")]` | — | 對應的更新日期欄位 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-updater` CSS 類別。
- 與 Creator 結構相同，但用途不同（記錄最後更新者而非建立者）。
