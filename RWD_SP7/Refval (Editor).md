# Refval (Editor)

> `EEPRWDTools.Core/Editors/Refval.cs` — 158 行
> 繼承：`RWDEditor`

## 用途

**參考值選取編輯器**（Reference Value）。提供彈出式查詢面板讓使用者從遠端資料中選取值，支援分頁、欄位對應回寫、模糊查詢等功能。是 EEP 中最常用的欄位查詢元件之一。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **valueTitle** | string | 文字 | — | 值欄位標題 |
| **textTitle** | string | 文字 | — | 文字欄位標題 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢面板顯示欄位 |
| **pageSize** | int | 數字框 `[NumberboxEditor]` | 10 | 每頁筆數 |
| **pageList** | JArray | 陣列編輯器 `[ArrayEditor]` | [] | 每頁筆數選項 |
| **validateXss** | bool | 核取方塊 | true | 是否驗證 XSS |
| **queryMode** | RefvalQueryMode | 列舉 | — | 查詢模式（none / fuzzy） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件 |
| **columnMatchs** | List\<ColumnMatch\> | 集合編輯器 `[CollectionEditor]` | [] | 欄位對應回寫 |
| **showValueText** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否同時顯示值和文字 |
| **checkData** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否驗證輸入值 |
| **autoQueryColumn** | bool | 核取方塊 | false | 是否自動查詢欄位 |
| **panelTitle** | string | 文字 `[Localization]` | — | 查詢面板標題（支援多語） |
| **selectOnly** | bool | 核取方塊 | false | 是否僅允許從面板選取 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **fit** | bool | `[Security("field")]` `[CheckboxEditor]` | false | 是否自適應寬度 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(row)]` | — | 選取事件（回傳整列資料） |

## 內部類別

| 類別 | 說明 |
|------|------|
| **Column** | 查詢面板欄位定義（Title、Field、Alignment、Sortable、Format） |
| **WhereItem** | 查詢條件項目（Field、Operator、Value）— 被 Combobox、Options 等共用 |
| **ColumnMatch** | 欄位對應回寫（TargetField → SourceField） |
| **Row / Constant / Varaible / Function / Parent** | WhereItem.Value 的不同值來源類型（實作 IValueType） |

## 備註

- `WhereItem` 是共用類別，Combobox、Options、Autocomplete、Submenu 都引用 `Refval.WhereItem`。
- `ColumnMatch` 可將選取列的其他欄位回寫到父容器的對應欄位。
- Column.Format 支援多種格式化：N0（數字）、日期、邏輯值、圖片、檔案、條碼、QRCode、地圖、簽名、鑽取、流程旗標等。
