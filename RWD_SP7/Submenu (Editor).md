# Submenu (Editor)

> `EEPRWDTools.Core/Editors/Submenu.cs` — 55 行
> 繼承：`RWDEditor`

## 用途

**階層式下拉選單編輯器**。支援樹狀結構的選項選取，透過 `parentField` 建立父子關係。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **parentField** | string | 欄位選擇器 `[ColumnEditor]` | — | 父層欄位（建立樹狀關係） |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onBeforeLoad** | string | 事件編輯器 `[ScriptEditor(param)]` | — | 載入前事件 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |

## 備註

- 渲染為 `<select>` 加上 `bootstrap-submenu` CSS 類別。
- 與 Combobox 的差異在於支援 `parentField` 做階層式選單。
