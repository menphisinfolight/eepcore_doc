# Autocomplete (Editor)

> `EEPRWDTools.Core/Editors/Autocomplete.cs` — 27 行
> 繼承：`RWDEditor`

## 用途

**自動完成輸入框編輯器**。使用者輸入時自動從遠端查詢並顯示建議列表，類似搜尋引擎的自動完成功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 10551–10620 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 直接回傳 `$(jq).val()` |
| `setValue(value)` | 直接設定 `$(jq).val(value)` |
| `readonly(bool)` | 設定 `disabled` 屬性 |
| `getWhereItems()` | 組合查詢條件，支援從 datagrid 列或 form 列取得動態值 |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **typeahead 整合**：初始化時呼叫 `target.typeahead()`，container 為 `body`。
- **遠端資料快取**：首次查詢透過 `$.loadData(remoteName, ...)` 取得建議清單，結果存入 `target.data('values')` 快取，後續輸入直接使用快取資料。
- **whereItems 動態條件**：查詢時可依 `whereItems` 設定的欄位和運算子組合過濾條件，值透過 `$.getDefaultValue()` 解析（支援表單/datagrid 當前列參照）。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-autocomplete` CSS 類別。
- 與 Refval 不同，Autocomplete 沒有彈出面板，直接在輸入框下方顯示建議。
