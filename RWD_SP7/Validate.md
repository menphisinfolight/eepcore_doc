# Validate

> `EEPRWDTools.Core/Controls/Validate.cs` — 117 行
> 繼承：`RWDControl` → `Component`

## 用途

**欄位驗證元件**（Validate）。

Validate 用來定義 DataGrid 或 DataForm 中欄位的前端驗證規則。支援必填檢查及多種驗證類型，包括長度、範圍、大小比較、格式（URL、Email）、身分證字號、統編、自訂函式、唯一值等。

## JSON 設定範例

```json
{
  "type": "validate",
  "id": "valOrder",
  "bindingObject": "gridOrder",
  "columns": [
    { "field": "ORDER_NO", "required": true, "validType": { "type": "maxlength", "length": 20 } },
    { "field": "EMAIL", "required": true, "validType": { "type": "email" } },
    { "field": "QTY", "required": false, "validType": { "type": "range", "from": "1", "to": "9999" } }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（datagrid, dataform） | — | 綁定的目標元件 ID |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 欄位驗證規則定義 |

### Column 子類別

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **title** | string | 文字 | — | 欄位標題 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 目標欄位名稱 |
| **required** | bool | 核取方塊 `[CheckboxEditor]` | `true` | 是否為必填欄位 |
| **validType** | string | 值選項編輯器 `[ValueOptionEditor]` | — | 驗證類型（依類型解析） |

### 驗證類型（IValueType）

| 類型 | 屬性 | 預設值 | 說明 |
|------|------|--------|------|
| **Length** | `from`, `to` | 0, 100 | 字串長度必須在指定範圍內 |
| **MinLength** | `length` | 3 | 最小長度 |
| **MaxLength** | `length` | 10 | 最大長度 |
| **Range** | `from`, `to` | "0", "100" | 數值範圍 |
| **Greater** | `value` | "0" | 必須大於指定值 |
| **Less** | `value` | "100" | 必須小於指定值 |
| **Url** | — | — | 驗證 URL 格式 |
| **Email** | — | — | 驗證 Email 格式 |
| **Cid** | — | — | 驗證身分證字號 |
| **Tid** | — | — | 驗證統一編號 |
| **Uid** | — | — | 驗證統一證號（外來人口） |
| **Function** | `name`, `message` | —, "Value is not valid." | 自訂驗證函式 |
| **Unique** | — | — | 唯一值檢查 |

## 備註

- `Render()` 方法為空實作，Validate 不產生可見的 HTML 元素，僅提供前端 JavaScript 使用的驗證規則。
- 一個欄位只能指定一種 `validType`，但 `required` 可與任何驗證類型並用。
- `Function` 類型允許指定自訂 JavaScript 函式名稱和驗證失敗訊息。
