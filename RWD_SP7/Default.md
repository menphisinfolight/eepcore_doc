# Default

> `EEPRWDTools.Core/Controls/Default.cs` — 64 行
> 繼承：`RWDControl` → `Component`

## 用途

**欄位預設值元件**（Default）。

Default 用來定義 DataGrid 或 DataForm 中欄位的預設值。當新增資料時，系統自動將指定欄位填入預設值。支援多種值類型：常數（Constant）、變數（Variable）、函式（Function）、父層欄位（Parent）。

## JSON 設定範例

```json
{
  "type": "default",
  "id": "defOrder",
  "bindingObject": "gridOrder",
  "columns": [
    { "field": "STATUS", "defaultValue": { "type": "constant", "value": "N" } },
    { "field": "CREATE_DATE", "defaultValue": { "type": "varaible", "value": "today" } },
    { "field": "DEPT_ID", "defaultValue": { "type": "parent", "field": "DEPT_ID" } },
    { "field": "SEQ", "defaultValue": { "type": "function", "name": "getSeq", "parameter": "" } }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（datagrid, dataform） | — | 綁定的目標元件 ID |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 欄位預設值定義 |

### Column 子類別

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **title** | string | 文字 | 欄位標題 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | 目標欄位名稱 |
| **defaultValue** | string | 值選項編輯器 `[ValueOptionEditor]` | 預設值（依類型解析） |
| **carryOn** | bool | 核取方塊 | 是否沿用上一筆值（連續新增時） |

### 值類型（IValueType）

| 類型 | 屬性 | 說明 |
|------|------|------|
| **Constant** | `value` | 固定常數值 |
| **Varaible** | `value`（`[DefaultValueEditor]`） | 系統變數（如 today、userId 等） |
| **Function** | `name`, `parameter` | 呼叫自訂函式取得值 |
| **Parent** | `field` | 取父層元件的指定欄位值 |

## 備註

- `Render()` 方法為空實作，Default 不產生可見的 HTML 元素，僅提供前端 JavaScript 使用的 data-options。
- `Varaible` 的拼寫為原始碼中的命名（非 Variable）。
- `CarryOn` 屬性適用於連續新增場景，保留上一筆的欄位值作為下一筆的預設值。
