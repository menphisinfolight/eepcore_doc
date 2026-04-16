# DataList

> `EEPRWDTools.Core/Controls/DataList.cs` — 131 行
> 繼承：`RWDControl` → `Component`

## 用途

**資料清單元件**（Data List）。

DataList 以卡片/清單樣式呈現資料列表，適用於 RWD 行動裝置或簡潔列表場景。支援分頁、篩選、檢視/編輯/刪除操作，以及多種前端事件。與 DataGrid 的表格形式不同，DataList 以 `<div>` 為基礎的響應式佈局呈現每筆資料。

## JSON 設定範例

```json
{
  "type": "datalist",
  "id": "listOrder",
  "remoteName": "cmdOrder",
  "title": "訂單清單",
  "pagination": true,
  "pageSize": 10,
  "bordered": true,
  "listCls": "col-xs-12",
  "editForm": "formOrder",
  "viewCommandVisible": true,
  "editCommandVisible": true,
  "deleteCommandVisible": false,
  "columns": [
    { "field": "ORDER_NO", "columnCls": "col-xs-12 col-sm-6" },
    { "field": "ORDER_DATE", "columnCls": "col-xs-12 col-sm-6" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源 RemoteName |
| **whereStr** | string | 文字 | — | 過濾條件字串 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 清單欄位定義 |
| **listCls** | string | 欄位樣式選擇器 `[ColumnClassEditor]` | `"col-xs-12"` | 清單容器的 CSS class |
| **viewCommandVisible** | bool | 核取方塊 | — | 是否顯示檢視按鈕（受安全性控制 `[Security]`） |
| **editCommandVisible** | bool | 核取方塊 | — | 是否顯示編輯按鈕（受安全性控制） |
| **deleteCommandVisible** | bool | 核取方塊 | — | 是否顯示刪除按鈕（受安全性控制） |
| **editForm** | string | 元件選擇器 `[ControlEditor]`（dataform） | — | 編輯表單元件 ID |
| **pagination** | bool | 核取方塊 `[CheckboxEditor]` | `true` | 是否啟用分頁 |
| **pageSize** | int | 數字框 `[NumberboxEditor]`（min=1, 預設=10） | 10 | 每頁筆數 |
| **title** | string | 文字（多語系） `[Localization]` | — | 清單標題 |
| **bordered** | bool | 核取方塊 `[CheckboxEditor]` | `true` | 是否顯示邊框 |

### 事件屬性

| 屬性 | 參數 | 說明 |
|------|------|------|
| **onBeforeLoad** | `(param)` | 載入前事件，可修改查詢參數 |
| **onLoad** | `(data)` | 載入完成事件 |
| **onSelect** | `(index, row)` | 選取資料列事件 |
| **onUpdate** | `(row)` → `return true;` | 更新前事件，回傳 false 可取消 |
| **onDelete** | `(row)` → `return true;` | 刪除前事件，回傳 false 可取消 |

### Column 子類別

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **field** | string | 欄位選擇器 `[ColumnEditor]` | 欄位名稱 |
| **columnCls** | string | 欄位樣式選擇器 `[ColumnClassEditor]` | 欄位 CSS class（預設 `"col-xs-12 col-sm-12"`） |
| **formatter** | string | 腳本編輯器 `[ScriptEditor]`（value, row, index） | 自訂格式化函式，預設 `return value;` |

### ToolItem 子類別

| 屬性 | 類型 | 說明 |
|------|------|------|
| **text** | string | 按鈕文字（多語系） |
| **iconCls** | string | 圖示 CSS class |
| **iconAlign** | IconAlign | 圖示對齊（Left/Right） |
| **btnCls** | string | 按鈕樣式 CSS class |
| **hidden** | bool | 是否隱藏（受安全性控制） |
| **onclick** | string | 點擊事件 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 8083–8387 行
> jQuery 外掛名稱：`$.fn.datalist`

### 公開 API 方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `options()` | — | 取得元件選項物件 |
| `init(options?)` | options | 初始化元件：解析 `data-options`、綁定事件、載入資料 |
| `bindEvent()` | — | 綁定 `.list-item` 的 click 事件，點擊時標記 `selected` 並觸發 `onSelect(index, row)` |
| `load(clear?)` | clear: boolean | 透過 `$.loadData` 向後端取得資料。`clear=true` 時重設頁碼並清空現有項目 |
| `loadData(data)` | data: `{rows, total}` | 將資料渲染成 DOM。支援兩種模式：`opts.rows > 0` 時以多列群組排列；否則逐筆建立 `.list-item` |
| `getRows()` | — | 回傳所有 `.list-item` 的 `data('row')` 陣列 |
| `getSelectedIndex()` | — | 回傳目前選取項目的 index（`.list-item.selected`） |
| `viewRow(index?)` | index | 以 `status:'view'` 開啟 editForm 表單 |
| `insert_row()` | — | 取得預設值後以 `status:'inserted'` 開啟 editForm。觸發 `onInsert` 事件（回傳 false 可取消） |
| `edit_row(index?)` | index | 以 `status:'updated'` 開啟 editForm。觸發 `onUpdate` 事件 |
| `delete_row(index?)` | index | 顯示確認對話框後透過 `$.updateData` 刪除該筆記錄，並從 DOM 移除。觸發 `onDelete` 事件 |
| `setWhere(where)` | Array 或 string | 設定 `whereItems`（陣列）或 `whereStr`（字串），然後重新載入資料（`load(true)`） |

### 關鍵前端行為

- **分頁載入**：啟用 `pagination` 時，若尚有更多頁面，會在清單底部產生「load more」按鈕，點擊後遞增 `page` 並再次呼叫 `load()`。
- **XSS 防護**：預設啟用 `validateXss`，渲染欄位值時會呼叫 `$.validateScript()` 過濾可能的腳本注入。
- **操作指令 Popover**：當 `viewCommandVisible` / `editCommandVisible` / `deleteCommandVisible` 啟用時，每筆項目右側顯示指令按鈕圖示，點擊後以 Bootstrap Popover 呈現檢視/編輯/刪除按鈕。
- **欄位格式化**：`columns[].formatter` 可自訂欄位渲染，呼叫簽名為 `formatter(value, row, index)`。
- **loadFilter**：`opts.loadFilter` 可在渲染前對 `data.rows` 進行資料過濾或轉換。

## 備註

- 渲染時輸出 `<div class="bootstrap-datalist clearfix">`，若 `Bordered` 為 true 則加上 `list-bordered` class。
- ToolItem 的 `Render()` 方法會產生 `<button>` 標籤，自動組合圖示與文字。
- 安全性屬性（`[Security]`）會依使用者權限自動控制按鈕的顯示/隱藏。
