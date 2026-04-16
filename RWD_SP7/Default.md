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

## 前端行為（JavaScript）

Default 元件本身不產生 HTML 元素，也沒有獨立的 `$.createObj`。其前端邏輯完全內嵌於 datagrid 和 form 的初始化／新增流程中。

### 核心機制

**`$.getDefaultValue(value, param)`**（全域工具函式，約第 686 行）：
- 解析 `defaultValue` 字串，根據規則名稱（`constant`、`varaible`、`function`、`parent`、`autoseq`、`row`）呼叫 `$.fn.defaultValue.defaults.rules` 中對應的處理函式。
- 以正規式 `/([a-zA-Z_]+)(.*)/` 拆分規則名稱與參數，參數部分透過 `eval()` 執行。

**`$.fn.defaultValue.defaults.rules`**（約第 14621 行）：
| 規則 | 行為 |
|------|------|
| `constant` | 直接回傳 `param[0]` |
| `varaible` | 依變數名稱回傳系統值：`today`（yyyy/MM/dd）、`todayc8`（yyyyMMdd）、`firstday`/`lastday`（當月首末日）、`now`（含時間）等；其餘從 `sessionStorage.clientInfo` 取值 |
| `function` | 呼叫 `$.callFunction(param[0], ...)` 執行自訂函式 |
| `parent` | 從 `$.fn.defaultValue.defaults.datagrid` 取得父層元件（form 或 datagrid），讀取指定欄位值 |
| `row` | 從 `$.fn.defaultValue.defaults.row` 取得當前列資料的指定欄位值 |
| `autoseq` | 掃描 datagrid/datalist 所有列，取目標欄位最大值後加上 step，不足位數補零（詳見 [[AutoSeq]]） |

### 呼叫時機

1. **DataGrid 新增列**（`insert_row`，約第 4641 行）：
   - 先設定 `$.fn.defaultValue.defaults.datagrid = $(this)`，提供 parent/autoseq 規則存取上下文。
   - 呼叫 `datagrid('getDefaultValues', parentRow)` 或 `form('getDefaultValues', parentRow)` 取得預設值物件。
   - 將預設值物件傳入 `onInsert` 事件，再作為新增列的初始資料。

2. **DataGrid `getDefaultValues(jq, values)`**（約第 5124 行）：
   - 遍歷所有欄位，依序處理：
     - 若欄位的 `carryOn` 為 `true` 且存在 `lastRow`（上一筆資料），沿用上一筆的值。
     - 否則呼叫 `$.getDefaultValue(options.defaultValue)` 解析預設值。
   - 額外支援 chatUX 參數：從 URL query string 的 `c=` 參數取得 sessionStorage 資料，解析 whereStr 填入對應欄位。

3. **Form 查詢面板初始化**（`initQuery`，約第 3196 / 6671 行）：
   - DataGrid 和 Form 的查詢面板初始化時，均呼叫 `form('getDefaultValues')` 填入預設值。

4. **Form 清除**（`clear` 按鈕，約第 6713 行）：
   - 清除時重新載入 `getDefaultValues()` 作為初始資料。

5. **複製列**（`copy_row`，約第 4793 行）：
   - 以被複製列的值作為 `values` 參數傳入 `getDefaultValues`，再覆寫 key 欄位。

### carryOn 處理

`carryOn` 在 `getDefaultValues` 中實作：若欄位設定 `carryOn: true`，且 datagrid 的 `lastRow`（透過 `$(this).data('lastRow')` 儲存的上一筆編輯列）存在且該欄位有值，直接沿用上一筆值，不走 defaultValue 規則。`lastRow` 在 `endEdit` 成功時設定（約第 4067 行）。

## 備註

- `Render()` 方法為空實作，Default 不產生可見的 HTML 元素，僅提供前端 JavaScript 使用的 data-options。
- `Varaible` 的拼寫為原始碼中的命名（非 Variable）。
- `CarryOn` 屬性適用於連續新增場景，保留上一筆的欄位值作為下一筆的預設值。
