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

## 備註

- 渲染時輸出 `<input type="hidden" class="bootstrap-fieldonblur">`，以隱藏元素存在於 DOM 中。
- 運算式中以大括號包裹欄位名稱來引用同一列中其他欄位的值。
- 當 Expression 中參考的任何欄位失去焦點時，目標欄位會自動重新計算。
