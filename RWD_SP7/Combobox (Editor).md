# Combobox (Editor)

> `EEPRWDTools.Core/Editors/Combobox.cs` — 65 行
> 繼承：`RWDEditor`

## 用途

**下拉選單編輯器**。支援靜態項目（Items）、遠端資料來源（RemoteName）、系統參數（FromSysParameters）三種資料模式。可多選，渲染為 `<select>` 標籤。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **fromSysParameters** | bool | 核取方塊 | false | 是否從系統參數載入選項 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | [] | 靜態選項清單（value + text） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onBeforeLoad** | string | 事件編輯器 `[ScriptEditor(param)]` | — | 載入前事件 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |
| **multiple** | bool | 核取方塊 | false | 是否允許多選 |
| **allowEmpty** | bool | `[Security("field")]` | false | 是否允許空值 |
| **multipleSeparator** | string | 文字 | "," | 多選時的分隔字元 |

## 備註

- 內部類別 `Item` 包含 `Value` 和 `Text` 兩個屬性。
- `WhereItems` 共用 `Refval.WhereItem` 類別，支援動態篩選條件。
