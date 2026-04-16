# Counter

> `EEPRWDTools.Core/Controls/Counter.cs` — 28 行
> 繼承：`RWDControl` → `Component`

## 用途

**計數器元件**。從遠端資料來源讀取資料，依指定的 Key 欄位和計數欄位顯示計數值。支援自訂格式化函式。

## JSON 設定範例

```json
{
  "type": "counter",
  "id": "cntVisitor",
  "remoteName": "cmdVisitorCount",
  "keyField": "CATEGORY",
  "counterField": "COUNT_VALUE"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源 |
| **keyField** | string | 欄位選擇器 `[ColumnEditor]` | — | 鍵值欄位名稱 |
| **counterField** | string | 欄位選擇器 `[ColumnEditor]` | — | 計數欄位名稱 |
| **formatter** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 格式化函式（預設回傳空字串） |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.counter`（`bootstrap.infolight.js` 第 19202–19317 行）

### 初始化流程

若 `keyValue` 已設定，初始化時立即呼叫 `load(keyValue)` 載入資料。

### 主要方法

| 方法 | 說明 |
|------|------|
| `load(keyValue)` | 根據 `keyField`（支援逗號分隔多欄位）組合 `whereItems` 條件，透過 `$.loadData(remoteName, ...)` 查詢資料。取得資料後以 `formatter(row)` 格式化並設為元素 HTML。 |
| `addValue(keyValue, callback)` | 先 `load` 取得現有資料，找到匹配列後將 `counterField` 值 +1（以 `updated` 更新）；若無匹配列則以 `counterField = 1` 新增（`inserted`）。透過 `$.updateData` 存回伺服端後重新 `load`。 |

### 資料操作細節

- `keyField` 與 `keyValue` 均支援逗號分隔的複合鍵（多欄位比對）。
- `addValue` 計數遞增邏輯：取得數值後 `Number(value) + 1`，NaN 時視為 0。
- 更新使用 `$.updateData(remoteName, datas, false, ...)`，`datas` 包含 `table`（從 remoteName 取 `.` 後半段）及 `updated` 或 `inserted` 陣列。

## 備註

- 渲染為 `<div class="bootstrap-counter">`，前端 JS 處理計數動畫與顯示邏輯。
- Formatter 可自訂每筆資料的顯示方式。
