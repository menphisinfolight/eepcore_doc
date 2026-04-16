# ClientMove

> `EEPRWDTools.Core/Controls/ClientMove.cs` — 137 行
> 繼承：`RWDControl` → `Component`

## 用途

**客戶端搬移元件**（Client Move）。

ClientMove 提供從來源資料（RemoteName）查詢並選取記錄，搬移至目標 DataGrid 的功能。支援查詢條件（WhereItems）、欄位對應（ColumnMatchs）、主鍵比對（KeyFields）、分頁，以及 XSS 驗證。

## JSON 設定範例

```json
{
  "type": "clientmove",
  "id": "cmProduct",
  "title": "選擇產品",
  "remoteName": "cmdProduct",
  "pageSize": 10,
  "pageList": [10, 20, 50],
  "height": 300,
  "targetDataGrid": "dgOrder",
  "alwaysInsert": false,
  "triggerExpression": true,
  "columns": [
    { "title": "品名", "field": "PROD_NAME", "format": "" }
  ],
  "whereItems": [
    { "field": "PROD_NO", "operator": "=", "value": "" }
  ],
  "keyFields": [
    { "targetField": "PROD_NO", "sourceField": "PROD_NO" }
  ],
  "columnMatchs": [
    { "targetField": "ITEM_NAME", "sourceField": "PROD_NAME" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 `[Localization]` | — | 對話框標題（支援多語系） |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 來源資料命令 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 顯示欄位集合 |
| **pageSize** | int | 數字框 `[NumberboxEditor]` | 10 | 每頁筆數 |
| **pageList** | JArray | 陣列編輯器 `[ArrayEditor("int")]` | `[]` | 可選的每頁筆數列表 |
| **validateXss** | bool | — | `true` | 是否驗證 XSS |
| **height** | int? | 數字框 `[NumberboxEditor]` | — | 表格高度（px） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | — | 查詢條件集合 |
| **alwaysInsert** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否永遠新增（不覆蓋既有列） |
| **triggerExpression** | bool | 勾選 `[CheckboxEditor]` | `true` | 搬移後是否觸發表達式計算 |
| **targetDataGrid** | string | 元件選擇器 `[ControlEditor]` | — | 目標 DataGrid 元件 ID |
| **keyFields** | List\<KeyField\> | 集合編輯器 `[CollectionEditor]` | — | 主鍵欄位對應（判斷重複） |
| **columnMatchs** | List\<ColumnMatch\> | 集合編輯器 `[CollectionEditor]` | — | 欄位值對應（來源→目標） |

### Column 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題（支援多語系） |
| **field** | string | 來源欄位名稱 |
| **format** | string | 顯示格式（N0 / yyyy/MM/dd / logic / checkbox / image / file / barcode / qrcode / map / signature / flowflag） |

### WhereItem 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **field** | string | 查詢欄位 |
| **operator** | string | 運算子（預設 `=`） |
| **value** | string | 值來源（支援 Constant / Variable / Function / Parent） |

### KeyField 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **targetField** | string | 目標 DataGrid 欄位 |
| **sourceField** | string | 來源資料欄位 |

### ColumnMatch 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **targetField** | string | 目標 DataGrid 欄位 |
| **sourceField** | string | 來源資料欄位 |
| **sourceValue** | string | 固定值來源（支援 Constant / Variable / Function / Parent） |

## 值類型（IValueType）

WhereItem.Value 和 ColumnMatch.SourceValue 支援多種值類型：

| 類型 | 說明 |
|------|------|
| **Constant** | 固定常數值 |
| **Varaible** | 系統變數（如登入帳號等） |
| **Function** | 函式呼叫（Name + Parameter） |
| **Parent** | 父層表單欄位值 |

## 備註

- Render 輸出 `<div class="bootstrap-clientmove">`，前端以對話框呈現來源資料供使用者勾選搬移。
- `targetDataGrid` 僅接受 `datagrid` 類型元件。
- `keyFields` 用於比對來源與目標是否已存在相同記錄，避免重複搬移。若 `alwaysInsert` 為 `true` 則跳過比對，一律新增。
- 類別宣告為 `class`（非 `public class`），存取範圍為 internal。
