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

## 備註

- 渲染為 `<div class="bootstrap-counter">`，前端 JS 處理計數動畫與顯示邏輯。
- Formatter 可自訂每筆資料的顯示方式。
