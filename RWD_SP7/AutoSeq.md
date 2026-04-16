# AutoSeq

> `EEPRWDTools.Core/Controls/AutoSeq.cs` — 36 行
> 繼承：`RWDControl` → `Component`

## 用途

**自動序號元件**（Auto Sequence）。

AutoSeq 用來在 DataGrid 或 DataForm 中自動為欄位產生前端序號。與 AutoNumber（Server 端、寫入資料庫）不同，AutoSeq 在前端即時產生流水號，適用於明細列表的項次編號。

## JSON 設定範例

```json
{
  "type": "autoseq",
  "id": "seqDetail",
  "bindingObject": "gridDetail",
  "field": "SEQ_NO",
  "numDig": 3,
  "startValue": 1,
  "step": 1
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（datagrid, dataform） | — | 綁定的目標元件 ID |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 序號目標欄位 |
| **numDig** | int | 數字框 `[NumberboxEditor]`（min=1, 預設=3） | 3 | 序號位數（不足補零） |
| **startValue** | int | 數字框 `[NumberboxEditor]`（min=0, 預設=1） | 1 | 起始值 |
| **step** | int | 數字框 `[NumberboxEditor]`（min=1, 預設=1） | 1 | 遞增步進 |

## 備註

- `Render()` 方法為空實作，AutoSeq 不產生可見的 HTML 元素。
- 類別宣告為 `internal`（`class AutoSeq`，非 `public class`），僅供組件內部使用。
- 產生的序號範例（numDig=3, startValue=1, step=1）：`001`, `002`, `003`...
