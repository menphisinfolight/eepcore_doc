# Options (Editor)

> `EEPRWDTools.Core/Editors/Options.cs` — 56 行
> 繼承：`RWDEditor`

## 用途

**選項按鈕編輯器**（Radio / Checkbox）。以按鈕組方式顯示選項，支援單選和多選模式。與 Combobox 類似但以平鋪按鈕呈現。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **fromSysParameters** | bool | 核取方塊 | false | 是否從系統參數載入選項 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | [] | 靜態選項清單（value + text） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件 |
| **multiple** | bool | 核取方塊 | false | 是否允許多選 |
| **showTextbox** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示文字輸入框（「其他」選項） |
| **separator** | string | 文字 | "," | 多選時的分隔字元 |
| **mode** | OptionsMode | 列舉 | — | 選項模式 |
| **orientation** | Orientation | 列舉 | — | 排列方向 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |

## 備註

- 共用 `Combobox.Item` 類別作為靜態選項。
- 渲染為 `<input>` 加上 `bootstrap-selectoptions` CSS 類別。
