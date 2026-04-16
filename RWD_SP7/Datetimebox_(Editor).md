# Datetimebox (Editor)

> `EEPRWDTools.Core/Editors/Datetimebox.cs` — 45 行
> 繼承：`RWDEditor`

## 用途

**日期時間選取編輯器**。結合日期和時間選取功能，Render 時將日期格式和時間格式合併為完整的 datetime 格式。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 下拉選項 `[ItemsEditor]` | "yyyy-mm-dd" | 日期格式 |
| **timeFormat** | string | 下拉選項 `[ItemsEditor]` | "hh:ii" | 時間格式（hh:ii:ss / hh:ii / hhiiss / hhii） |
| **dataType** | DateboxDataType | 列舉 | — | 資料類型 |
| **selectOnly** | bool | 核取方塊 | false | 是否僅能從日曆選取 |
| **pickerPosition** | string | 下拉選項 `[ItemsEditor]` | — | 選取器位置 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(date)]` | — | 選取事件 |
| **minView** | DatetimeboxView | 列舉 | — | 最小檢視層級 |

## 前端行為（JavaScript）

Datetimebox 在 datagrid editors 中有自己的定義（`$.fn.datagrid.defaults.editors.datetimebox`），但**底層完全委派 `$.fn.datebox` 元件**。

### Editor 初始化（init）

1. 建立 `<input class="form-control"/>` 並附加到容器。
2. 複製 options 後進行**格式合併**：
   - 取得基礎格式 `opts.format`（預設 `'yyyy-mm-dd'`）。
   - 使用正則 `/(h{1,2}|H{1,2}).*i{1,2}/i` 檢測格式中是否已包含時間部分。
   - 若無時間部分且有 `opts.timeFormat`，則將時間格式附加到日期格式後（如 `'yyyy-mm-dd hh:ii'`）。
   - 將格式中的 `ss` 替換為 `00`（秒固定為零）。
   - 設定 `opts.minView` 預設為 `'day'`（允許選到日期+時間層級）。
3. 呼叫 `input.datebox('init', opts)` 委派給 datebox 元件。

### 底層 datebox 元件（`$.fn.datebox`）

`$.fn.datebox` 是透過 `$.createObj('datebox', ...)` 建立的完整 jQuery plugin，主要行為：

- **UI 結構**：將 `<input>` 包裹在 `div.input-group` 中，右側加上日曆圖示按鈕（`glyphicon-calendar`）。
- **日期選取器**：內部呼叫 `datetimepicker` 第三方套件，設定語系、格式、`minView`、`autoclose` 等。
- **位置計算**：覆寫 `place` 方法，當選取器超出視窗範圍時改為固定定位（居中顯示）並加上 `datebox-backdrop` 遮罩。
- **事件綁定**（`bindEvent`）：
  - 點擊日曆按鈕或 focus 時開啟 datetimepicker。
  - `change` 事件觸發 `onSelect` 回呼。
  - `selectOnly` 為 true 時設定 input 為 readOnly。

### getValue / setValue

- `getValue`：透過 `datebox('getValue')` 取值，支援 `varchar8` 格式轉換（如 `20260416` ↔ `2026-04-16`）。
- `setValue`：透過 `datebox('setValue')` 設值，會將 ISO 格式（含 T/Z）解析後依格式顯示。

### readonly

透過 `datebox('readonly')` 同時 disable input 和日曆按鈕。

### 工具方法

| 方法 | 說明 |
|------|------|
| `dateToChar8` | 日期字串去除分隔符（`2026-04-16` → `20260416`） |
| `char8ToDate` | 8 碼字串轉日期（`20260416` → `2026-04-16`） |
| `getDateValue` | 依格式重新排列日期各欄位為 `yyyy/mm/dd` 標準順序 |

## 備註

- Render 時 Format 會被合併為 `"{日期格式} {時間格式}"`（如 `"yyyy-mm-dd hh:ii"`）。
- 共用 Datebox 的 `bootstrap-datebox` CSS 類別。
