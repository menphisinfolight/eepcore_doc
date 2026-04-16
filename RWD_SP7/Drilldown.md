# Drilldown

> `EEPRWDTools.Core/Controls/Drilldown.cs` — 80 行
> 繼承：`RWDControl` → `Component`

## 用途

**鑽取元件**（Drilldown）。

Drilldown 綁定到 DataGrid，允許使用者點擊欄位值後鑽取顯示關聯的明細資料。支援多種開啟模式（Tab / Dialog / Window / Table），也可導向其他頁面。

## JSON 設定範例

```json
{
  "type": "drilldown",
  "id": "ddOrderDetail",
  "bindingObject": "dgOrder",
  "mode": "Dialog",
  "targetRemoteName": "cmdOrderDetail",
  "pageTitle": "訂單明細",
  "pageOpenForm": true,
  "columns": [
    { "field": "ORDER_NO", "operator": "=", "targetField": "ORDER_NO" }
  ],
  "targetColumns": [
    { "title": "品名", "field": "ITEM_NAME", "alignment": "left", "sortable": true }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]` | — | 綁定的 DataGrid 元件 ID |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 鑽取條件欄位對應 |
| **mode** | DrilldownMode | — | — | 開啟模式（Tab / Dialog / Window / Table） |
| **targetRemoteName** | string | 命令選擇器 `[CommandEditor]` | — | 目標資料命令 |
| **targetColumns** | List\<TargetColumn\> | 集合編輯器 `[CollectionEditor]` | — | 目標顯示欄位集合 |
| **page** | string | 選單編輯器 `[MenuEditor]` | — | 導向頁面 |
| **pageTitle** | string | 文字 `[Localization]` | `"Drilldown"` | 頁面標題（支援多語系） |
| **pageOpenForm** | bool | 勾選 `[CheckboxEditor]` | `true` | 導向頁面時是否開啟表單 |
| **onFormat** | string | 腳本編輯器 `[ScriptEditor]` | — | 格式化事件，簽名 `function(value, row){ return value; }` |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊事件，簽名 `function(row, whereItems){ return true; }` |

### Column 子項目（鑽取條件）

| 屬性 | 類型 | 說明 |
|------|------|------|
| **field** | string | 來源 DataGrid 欄位 |
| **operator** | string | 運算子（預設 `=`） |
| **targetField** | string | 目標資料欄位 |

### TargetColumn 子項目（目標欄位）

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題（支援多語系） |
| **field** | string | 目標欄位名稱 |
| **alignment** | string | 對齊方式（left / center / right） |
| **sortable** | bool | 是否可排序（預設 `false`） |
| **format** | string | 顯示格式（N0 / yyyy/MM/dd / logic / checkbox / image / file / barcode / qrcode / map / signature / drilldown / flowflag / creator / updater） |

## 備註

- Render 輸出 `<div class="bootstrap-drilldown">`。
- `bindingObject` 僅接受 `datagrid` 類型元件。
- 當 `mode` 為 `Table` 時，鑽取結果直接以表格形式顯示在當前頁面；其他模式則以分頁/對話框/新視窗開啟。
- TargetColumn 的 `format` 支援 `drilldown`，可實現多層鑽取。
- 類別宣告為 `class`（非 `public class`），存取範圍為 internal。
