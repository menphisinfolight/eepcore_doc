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

## 前端行為（JavaScript）

AutoSeq 元件本身不產生 HTML 元素，也沒有獨立的 `$.createObj`。其前端邏輯完全透過 Default 元件的預設值規則機制實現——C# 端將 AutoSeq 的設定轉換為 `defaultValue` 格式（`autoseq['欄位名', 位數, 起始值, 步進]`），交由 `$.fn.defaultValue.defaults.rules.autoseq` 處理。

### 核心函式

**`$.fn.defaultValue.defaults.rules.autoseq(param)`**（`bootstrap.infolight.js` 約第 14662 行）：

```js
autoseq: function(param) {
    // param[0] = 欄位名, param[1] = numDig, param[2] = startValue, param[3] = step
    var datagrid = $.fn.defaultValue.defaults.datagrid;
    var datalist = $.fn.defaultValue.defaults.datalist;
    var rows = datagrid ? datagrid.datagrid('getRows') : 
               datalist ? datalist.datalist('getRows') : null;
    if (rows) {
        var value = -1;
        for (var i = 0; i < rows.length; i++) {
            var fieldValue = parseInt(rows[i][param[0]]);
            if (!isNaN(fieldValue)) value = Math.max(fieldValue, value);
        }
        value = value < 0 ? param[2].toString() : (value + param[3]).toString();
        for (var i = value.length; i < param[1]; i++) value = '0' + value;
        return value;
    }
}
```

### 運作邏輯

1. 從 `$.fn.defaultValue.defaults.datagrid`（或 `datalist`）取得目前所有列資料。
2. 遍歷所有列，以 `parseInt()` 解析目標欄位值，取最大值。
3. 若所有列都無有效數值（`value` 仍為 -1），使用 `startValue` 作為起始值。
4. 否則以最大值加上 `step` 作為新序號。
5. 不足 `numDig` 位數時，前方補零。

### 呼叫時機

與 Default 元件相同——在 `insert_row`（DataGrid 新增列）時，`$.fn.defaultValue.defaults.datagrid` 被設定為當前 datagrid 後，`getDefaultValues` 遍歷欄位時呼叫 `$.getDefaultValue(options.defaultValue)` 觸發 `autoseq` 規則。

### 注意事項

- AutoSeq 是**純前端序號**，不會與 Server 端溝通。序號根據目前已載入的列資料計算，若分頁僅載入部分資料，可能產生重複序號。
- 支援 datagrid 與 datalist 兩種元件。

## 備註

- `Render()` 方法為空實作，AutoSeq 不產生可見的 HTML 元素。
- 類別宣告為 `internal`（`class AutoSeq`，非 `public class`），僅供組件內部使用。
- 產生的序號範例（numDig=3, startValue=1, step=1）：`001`, `002`, `003`...
