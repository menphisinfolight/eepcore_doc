# Autocomplete (Editor)

> `EEPRWDTools.Core/Editors/Autocomplete.cs` — 27 行
> 繼承：`RWDEditor`

## 用途

**自動完成輸入框編輯器**。使用者輸入時自動從遠端查詢並顯示建議列表，類似搜尋引擎的自動完成功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-autocomplete` CSS 類別。
- 與 Refval 不同，Autocomplete 沒有彈出面板，直接在輸入框下方顯示建議。
