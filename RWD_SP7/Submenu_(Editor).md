# Submenu (Editor)

> `EEPRWDTools.Core/Editors/Submenu.cs` — 55 行
> 繼承：`RWDEditor`

## 用途

**階層式下拉選單編輯器**。支援樹狀結構的選項選取，透過 `parentField` 建立父子關係。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **parentField** | string | 欄位選擇器 `[ColumnEditor]` | — | 父層欄位（建立樹狀關係） |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onBeforeLoad** | string | 事件編輯器 `[ScriptEditor(param)]` | — | 載入前事件 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 13931–14105 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 從 `data('value')` 取得已選值 |
| `setValue(value)` | 設定值並更新按鈕顯示文字；找不到對應項目時顯示「請選擇」 |
| `readonly(bool)` | 對 dropdown-toggle 按鈕設定/移除 `disabled` |
| `load()` | 透過 `$.loadData(remoteName, ...)` 載入遠端資料 |
| `loadData(rows)` | 將平面資料依 `parentField` 遞迴建構樹狀選單 HTML |
| `getWhereItems()` | 組合查詢條件，透過 `$.getDefaultValue()` 解析動態值 |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **Bootstrap Dropdown 結構**：初始化時隱藏原始 `<select>`，外層包裝為 `.dropdown.bootstrap-submenu`，建立 toggle 按鈕及 `<ul class="dropdown-menu">`。
- **樹狀選單建構**：`loadData` 以 `parentField` 遞迴分組，有子項目的節點建立 `dropdown-submenu` 巢狀結構，呼叫 `$('[data-submenu]').submenupicker()` 啟用子選單行為。
- **onRenderNode 自訂顯示**：若設定 `onRenderNode` 回呼，可自訂節點顯示文字。
- **事件**：點擊葉節點時更新值與按鈕文字，觸發 `onSelect` 回呼；`onBeforeLoad` 可在載入前修改查詢參數。
- **預設欄位**：`parentField` 預設 `parentId`，`idField` / `valueField` 預設 `id`，`textField` 預設 `text`。

## 備註

- 渲染為 `<select>` 加上 `bootstrap-submenu` CSS 類別。
- 與 Combobox 的差異在於支援 `parentField` 做階層式選單。
