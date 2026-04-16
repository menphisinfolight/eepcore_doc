# Refval (Editor)

> `EEPRWDTools.Core/Editors/Refval.cs` — 158 行
> 繼承：`RWDEditor`

## 用途

**參考值選取編輯器**（Reference Value）。提供彈出式查詢面板讓使用者從遠端資料中選取值，支援分頁、欄位對應回寫、模糊查詢等功能。是 EEP 中最常用的欄位查詢元件之一。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **valueTitle** | string | 文字 | — | 值欄位標題 |
| **textTitle** | string | 文字 | — | 文字欄位標題 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢面板顯示欄位 |
| **pageSize** | int | 數字框 `[NumberboxEditor]` | 10 | 每頁筆數 |
| **pageList** | JArray | 陣列編輯器 `[ArrayEditor]` | [] | 每頁筆數選項 |
| **validateXss** | bool | 核取方塊 | true | 是否驗證 XSS |
| **queryMode** | RefvalQueryMode | 列舉 | — | 查詢模式（none / fuzzy） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件 |
| **columnMatchs** | List\<ColumnMatch\> | 集合編輯器 `[CollectionEditor]` | [] | 欄位對應回寫 |
| **showValueText** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否同時顯示值和文字 |
| **checkData** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否驗證輸入值 |
| **autoQueryColumn** | bool | 核取方塊 | false | 是否自動查詢欄位 |
| **panelTitle** | string | 文字 `[Localization]` | — | 查詢面板標題（支援多語） |
| **selectOnly** | bool | 核取方塊 | false | 是否僅允許從面板選取 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **fit** | bool | `[Security("field")]` `[CheckboxEditor]` | false | 是否自適應寬度 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(row)]` | — | 選取事件（回傳整列資料） |

## 內部類別

| 類別 | 說明 |
|------|------|
| **Column** | 查詢面板欄位定義（Title、Field、Alignment、Sortable、Format） |
| **WhereItem** | 查詢條件項目（Field、Operator、Value）— 被 Combobox、Options 等共用 |
| **ColumnMatch** | 欄位對應回寫（TargetField → SourceField） |
| **Row / Constant / Varaible / Function / Parent** | WhereItem.Value 的不同值來源類型（實作 IValueType） |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 10117–10549 行（`$.fn.refval`）

### 渲染結構

`init` 將 `<input>` 包裝為：

```html
<div class="row">
  <div class="{valueCls|col-xs-12}">
    <div class="input-group">
      <input ...原始 input... />
      <span class="input-group-btn">
        <button class="btn btn-default form-btn glyphicon glyphicon-search"></button>
      </span>
    </div>
  </div>
  <div class="{textCls|hide}">
    <p class="form-control-static refval-text"></p>
  </div>
</div>
```

- `valueCls` / `textCls` 控制值欄位與文字欄位的 Bootstrap grid class。
- 若 `selectOnly == true`，input 會被設為 `readOnly`。

### 公開 API 方法

| 方法 | 說明 |
|------|------|
| `$el.refval('options')` | 取得元件選項物件 |
| `$el.refval('getValue')` | 取得目前值（`data('value')`） |
| `$el.refval('setValue', val)` | 設定值並透過 `$.getDisplayRow` 非同步載入顯示文字 |
| `$el.refval('setWhere', where)` | 設定篩選條件（字串型式設定 `whereStr`） |
| `$el.refval('readonly', bool)` | 設定唯讀狀態（disabled input 與搜尋按鈕） |
| `$el.refval('openModal')` | 手動開啟查詢彈窗 |
| `$el.refval('getWhereItems')` | 組合 whereItems 陣列（含搜尋關鍵字與設計時條件） |
| `$el.refval('doColumnMatch', row)` | 根據 `columnMatchs` 設定將 row 中的欄位值回寫至目標欄位 |

### 關鍵行為

**1. 搜尋按鈕 / Focus 開啟彈窗**
- 點擊搜尋按鈕或 focus 按鈕時，呼叫 `openModal` 開啟 Bootstrap Modal。
- 使用 `focus` CSS class 防止重複開啟。

**2. Enter 鍵觸發帶搜尋文字的彈窗**
- 在 input 按 Enter 時，將目前輸入存入 `data('search')`，清空 input 後開啟 Modal。
- `getWhereItems` 會將 `data('search')` 組為 `operator: '%'`（前方模糊）的篩選條件傳給 datagrid。

**3. Blur 驗證與顯示文字**
- 失焦時以 `$.getDisplayRow` 向後端查詢該值對應的完整 row。
- 若 `checkData == true` 且查無資料，顯示錯誤訊息「{欄位標題}:'{值}' 不存在」。
- 若 `textCls` 為 `hide`（文字欄隱藏），input 顯示為 `value:text` 或純 `text`（依 `showValueText`）。
- Focus 時還原顯示為原始 value。

**4. 彈窗（openModal）**
- 動態建立 Modal，內含 datagrid（`refval-table`）。
- 若無自訂 `columns`，自動用 `valueField` + `textField` 產生欄位。
- Column 支援 `format` 屬性，自動套用 `datagrid.formatValue`。
- datagrid 設定：`pagination: true`、`pageSize`、`pageList`、`whereStr`、`validateXss`、`autoQueryColumn`。
- 選取列後：設定 value、更新 refval-text、執行 `doColumnMatch`、觸發 `onSelect`、關閉 Modal。
- `queryMode === 'fuzzy'` 時，Modal 上方會出現額外的模糊搜尋輸入框，以所有 column 的 field 組成 OR 條件查詢（key 欄位用 `%` 前方比對，非 key 用 `%%` 模糊比對）。

**5. columnMatchs 自動回寫（doColumnMatch）**
- 遍歷 `opts.columnMatchs` 陣列，將 `row[sourceField]` 寫入 `targetField` 對應的編輯器。
- 支援 datagrid 內嵌編輯模式（透過 `editor.target` 取得 cell editor）與 form 模式（透過 `name` 或 `id` 查找）。
- 若目標欄位本身也是 refval，會呼叫 `refval('setValue')` 並觸發 blur 以連鎖解析。
- 每次回寫後觸發 `$.triggerValueChanged`，通知其他依賴元件。

**6. setValue 非同步載入**
- `setValue` 設定值後，延遲 100ms 呼叫 `$.getDisplayRow`（async=true）載入顯示文字。
- 延遲的目的是確保 whereItems 中引用的其他欄位已完成設值。

## 備註

- `WhereItem` 是共用類別，Combobox、Options、Autocomplete、Submenu 都引用 `Refval.WhereItem`。
- `ColumnMatch` 可將選取列的其他欄位回寫到父容器的對應欄位。
- Column.Format 支援多種格式化：N0（數字）、日期、邏輯值、圖片、檔案、條碼、QRCode、地圖、簽名、鑽取、流程旗標等。
