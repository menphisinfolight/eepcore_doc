# Combobox (Editor)

> `EEPRWDTools.Core/Editors/Combobox.cs` — 65 行
> 繼承：`RWDEditor`

## 用途

**下拉選單編輯器**。支援靜態項目（Items）、遠端資料來源（RemoteName）、系統參數（FromSysParameters）三種資料模式。可多選，渲染為 `<select>` 標籤。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **fromSysParameters** | bool | 核取方塊 | false | 是否從系統參數載入選項 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | [] | 靜態選項清單（value + text） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件（共用 Refval.WhereItem） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onBeforeLoad** | string | 事件編輯器 `[ScriptEditor(param)]` | — | 載入前事件 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |
| **multiple** | bool | 核取方塊 | false | 是否允許多選 |
| **allowEmpty** | bool | `[Security("field")]` | false | 是否允許空值 |
| **multipleSeparator** | string | 文字 | "," | 多選時的分隔字元 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 9505–9723 行（`$.fn.combobox`）

### 渲染結構

`init` 根據 `multiple` 屬性產生兩種模式：

- **單選模式**：為 `<select>` 加上 `form-select` CSS class，直接作為原生下拉選單。
- **多選模式**：設定 `multiple` 屬性後，初始化 Bootstrap `selectpicker` 外掛，提供多選 UI。`multipleSeparator`（預設 `,`）用於值的合併與拆分。

### 公開 API 方法

| 方法 | 說明 |
|------|------|
| `$el.combobox('options')` | 取得元件選項物件 |
| `$el.combobox('getValue')` | 取得目前值；多選時以 `multipleSeparator` 合併為字串 |
| `$el.combobox('setValue', val)` | 設定值；多選時自動以 `multipleSeparator` 拆分並觸發 change |
| `$el.combobox('load')` | 重新載入選項資料（遠端或靜態） |
| `$el.combobox('loadData', data)` | 以資料陣列填充 `<option>` 元素 |
| `$el.combobox('setWhere', where)` | 設定篩選條件並自動重新 `load` |
| `$el.combobox('getWhereItems')` | 組合 whereItems 陣列（解析動態預設值） |
| `$el.combobox('readonly', bool)` | 設定唯讀狀態（disabled） |

### 關鍵行為

**1. 三種資料載入模式（load）**

| 優先順序 | 模式 | 條件 | 行為 |
|----------|------|------|------|
| 1 | 系統參數 | `fromSysParameters == true` | 自動設定 `remoteName = 'SystemTable.sysParas'`，以欄位名稱為篩選條件（`COLUMNNAME = {field}`），valueField/textField 皆為 `VALUE` |
| 2 | 遠端資料 | `remoteName` 有值 | 透過 `$.loadData` 呼叫後端 API，支援 `whereStr` 與 `whereItems` 篩選 |
| 3 | 靜態項目 | `items` 有值 | 直接載入 `items` 陣列，valueField/textField 固定為 `value`/`text` |

**2. loadData 選項渲染**
- 清空現有 `<option>` 後，先插入一筆隱藏的「請選擇」提示項。
- 若 `allowEmpty == true`，在第一筆資料前插入空值選項。
- 遍歷 data 陣列產生 `<option value="{valueField}">{textField}</option>`。

**3. 多選值處理**
- 載入完成後，多選模式呼叫 `selectpicker('refresh')` 更新 UI。
- 還原值時依序嘗試：目前 `val()` → `data('value')`（可能為 array 或以 separator 分隔的字串）→ 空值。
- `getValue` 以 `multipleSeparator` join 回傳字串；`setValue` 以 `multipleSeparator` split 後設定。

**4. WhereItems 動態篩選**
- `getWhereItems` 解析 `whereItems` 中的值，透過 `$.getDefaultValue` 取得動態預設值（可引用同表單或同 datagrid 列的其他欄位值）。
- `setWhere` 重設條件後立即呼叫 `load` 重新載入資料，適用於主從連動場景。

**5. change 事件**
- 值變更時，將 `val()` 存入 `data('value')`（修正空字串 bug），並觸發 `onSelect` 回呼。

## 備註

- 內部類別 `Item` 包含 `Value` 和 `Text` 兩個屬性。
- `WhereItems` 共用 `Refval.WhereItem` 類別，支援動態篩選條件。
