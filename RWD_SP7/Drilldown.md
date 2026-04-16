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

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 13218–13432 行
> jQuery 外掛名稱：`$.fn.drilldown`

### 公開 API 方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `options()` | — | 取得元件選項物件 |
| `init(options?)` | options | 初始化元件並隱藏（`hide()`），元件本身不渲染可見內容 |
| `createLink(param)` | `{value, row, selector}` | 產生鑽取超連結 `<a>`。由 DataGrid 的 formatter 呼叫，點擊時觸發 `drilldown('click', this)`。支援 `onFormat` 自訂顯示文字 |
| `click(target)` | target: DOM | 從被點擊的 `<a>` 元素反推所屬 DataGrid 及列索引（支援多列 `trLength`），取得 row 後呼叫 `open(row)` |
| `openModal(whereItems)` | Array | `mode:'table'` 時使用：建立 Modal 對話框，內嵌 DataGrid 並以 `targetColumns` + `targetRemoteName` 載入鑽取資料，支援分頁（pageSize=10） |
| `open(row)` | row: object | 依據 `columns` 組合 `whereItems`，根據 `mode` 決定開啟方式 |

### 開啟模式行為

| mode | 行為 |
|------|------|
| `table` | 呼叫 `openModal()`，在 Modal 中以 DataGrid 表格顯示鑽取結果 |
| `tab` | 呼叫 `$.openTab(title, url)`，在主框架新增頁籤開啟目標頁面 |
| `dialog` | 呼叫 `$.openForm(title, url, {dialogStyle})`，以對話框開啟目標頁面 |
| 其他（預設） | 使用 `window.open(url, '_blank')` 在新視窗開啟 |

### 關鍵前端行為

- **鑽取參數傳遞**：將 `whereItems` 與 `drillRow` 序列化為 JSON 存入 `sessionStorage`，並透過 URL 參數 `?drill=<id>` 傳遞給目標頁面。
- **權限檢查**：當設定 `checkMenuRight` 時，開啟前會透過 AJAX POST `../main?mode=getMenus` 遞迴檢查使用者是否有目標頁面的存取權限。無權限時顯示 `accessDenied` 錯誤。
- **pageOpenForm**：啟用時會在鑽取參數中加入 `loadAction:'viewRow'`，讓目標頁面自動以檢視模式開啟表單。
- **dialogStyle**：`openModal` 支援 `large`（大尺寸 Modal）與 `fit`（寬度自適應、margin=0）樣式。
- **onClick 事件**：在 `open()` 中觸發，回傳 `false` 可取消鑽取操作。

## 備註

- Render 輸出 `<div class="bootstrap-drilldown">`。
- `bindingObject` 僅接受 `datagrid` 類型元件。
- 當 `mode` 為 `Table` 時，鑽取結果直接以表格形式顯示在當前頁面；其他模式則以分頁/對話框/新視窗開啟。
- TargetColumn 的 `format` 支援 `drilldown`，可實現多層鑽取。
- 類別宣告為 `class`（非 `public class`），存取範圍為 internal。
