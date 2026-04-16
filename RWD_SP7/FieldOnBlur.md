# FieldOnBlur

> `EEPRWDTools.Core/Controls/FieldOnBlur.cs` — 44 行
> 繼承：`RWDControl` → `Component`

## 用途

**欄位失焦運算元件**（Field On Blur）。

FieldOnBlur 用來定義當 DataGrid 或 DataForm 中的欄位失去焦點（blur）時，自動執行運算式並將結果寫入目標欄位。常見用途包括自動計算小計（數量 x 單價）、合計等。

## JSON 設定範例

```json
{
  "type": "fieldonblur",
  "id": "blurCalc",
  "bindingObject": "gridDetail",
  "columns": [
    { "targetField": "AMOUNT", "expression": "{QTY} * {PRICE}" },
    { "targetField": "TAX", "expression": "{AMOUNT} * 0.05" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（datagrid, dataform） | — | 綁定的目標元件 ID |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 運算欄位定義 |

### Column 子類別

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **title** | string | 文字 | 欄位標題 |
| **targetField** | string | 欄位選擇器 `[ColumnEditor]` | 運算結果寫入的目標欄位 |
| **expression** | string | 文字 | 運算式（以 `{欄位名}` 引用其他欄位值） |

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 14914–15064 行
> jQuery 外掛名稱：`$.fn.fieldonblur`

### 初始化流程

1. 解析 `data-options`（含 `bindingObject`、`columns` 陣列），儲存至元件 data。
2. 初始化時不主動觸發運算，需由外部呼叫 `trigger` 方法。

### 核心邏輯：trigger(name)

當某個欄位名稱 `name` 失焦時被呼叫，執行以下流程：

1. **比對觸發欄位**：遍歷 `columns`，以正規式從每個 `expression` 中提取所有欄位名稱（支援中文、英文、數字、底線、點號）。若提取結果包含 `name`，則該 column 需要重新計算。
2. **取得資料列**：依 `bindingObject` 類型取得當前資料列——DataGrid 使用 `datagrid('getRow')`，DataForm 使用 `form('getRow')`。
3. **建構動態函式**：為 expression 中的每個欄位建立 JavaScript 變數宣告：
   - `{元件ID}.Total.{欄位}` 格式：取 DataGrid 的 Total 列。
   - `{元件ID}.Row.{欄位}` 格式：取 DataForm 的整列資料。
   - 一般欄位：從當前資料列取值。若值為數字則宣告為 `var field = number`；否則宣告為字串（含跳脫處理）。
   - 特殊判斷：`0` 開頭的數字字串、科學記號格式（`xExx`）視為字串而非數字。
4. **執行運算**：以 `new Function(scripts.join(';'))` 動態建立函式並執行，取得計算結果。若執行失敗，顯示錯誤訊息。
5. **寫入目標欄位**：
   - `targetField` 含 `.`（如 `gridId.fieldName`）：將值寫入指定 DataGrid 的所有列。若該列處於編輯狀態，透過 editor 的 `setValue` 更新；否則使用 `datagrid('updateRow')` 更新。
   - `targetField` 不含 `.`：將值寫入 `bindingObject`。若為 DataGrid，更新編輯列的 editor；若為 DataForm，使用 `setValue` 設定。
6. **連鎖觸發**：目標欄位名稱加入 `triggerNames` 陣列，後續 column 若引用此欄位也會被觸發，形成連鎖計算。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化元件 |
| `trigger(name)` | 以欄位名稱觸發運算，執行相關 expression 並更新目標欄位 |

## 備註

- 渲染時輸出 `<input type="hidden" class="bootstrap-fieldonblur">`，以隱藏元素存在於 DOM 中。
- 運算式中以大括號包裹欄位名稱來引用同一列中其他欄位的值。
- 當 Expression 中參考的任何欄位失去焦點時，目標欄位會自動重新計算。
