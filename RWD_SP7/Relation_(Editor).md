# Relation (Editor)

> `EEPRWDTools.Core/Editors/Relation.cs` — 28 行
> 繼承：`RWDEditor`

## 用途

**關聯欄位編輯器**。用於顯示關聯資料表的欄位值，類似 Refval 但僅做值對應顯示，不提供查詢面板。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **separator** | string | 文字 | — | 分隔字元 |

## 備註

- `Render()` 方法為空實作，表示此編輯器不直接產生 HTML，可能由前端另行處理。
