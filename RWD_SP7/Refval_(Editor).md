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

## JavaScript 範例

### 1. 基本取值 / 設值（最常用）

```javascript
// 取得目前值
var userCode = $('#dfMaster_USER_CODE').refval('getValue');

// 設定值（會自動非同步向後端撈 textField 顯示文字，100ms delay 後執行）
$('#dfMaster_USER_CODE').refval('setValue', 'U001');

// 清空值
$('#dfMaster_USER_CODE').refval('setValue', '');

// 切換唯讀
$('#dfMaster_USER_CODE').refval('readonly', true);   // 鎖住 input + 搜尋按鈕
$('#dfMaster_USER_CODE').refval('readonly', false);  // 解鎖
```

> **ID 慣例**：在 DataForm 中，欄位 ID 為 `{formId}_{欄位名稱}`（例如 `dfMaster_USER_CODE`）。DataPanel 內的欄位則直接用 `{欄位名稱}` 的 ID。

### 2. setValue 的非同步行為要注意

`setValue` 不是同步的！內部會 delay 100ms 後呼叫 `$.getDisplayRow` 向後端撈顯示文字：

```javascript
// ❌ 錯誤：setValue 後立刻拿 text 會拿不到
$('#dfMaster_USER_CODE').refval('setValue', 'U001');
var text = $('#dfMaster_USER_CODE').closest('.row').find('.refval-text').text();
console.log(text);  // 空字串，因為顯示文字還沒載入

// ✅ 正確：要拿 text 用 $.getDisplayRow 同步查詢
var opts = $('#dfMaster_USER_CODE').refval('options');
$.getDisplayRow(opts, 'U001', function(row) {
    console.log(row);         // { USER_CODE: 'U001', USER_NAME: '張三', ... }
    console.log(row.USER_NAME);
}, false);  // false = 同步
```

### 3. onSelect 事件：取得完整選取列

```javascript
// 在設計介面設 onSelect 事件方法名稱（例如 onUserSelect），對應的 JS 如下：
function onUserSelect(row) {
    // row 是選取的完整資料列（包含所有查詢面板的欄位）
    if (!row) return;  // 清空時 row = {}

    console.log('使用者編號：' + row.USER_CODE);
    console.log('使用者姓名：' + row.USER_NAME);
    console.log('部門代碼：' + row.DEPT_CODE);

    // 可以在這裡做業務邏輯，例如依選的使用者載入其訂單
    $('#dgOrders').datagrid('reload', 1);
}
```

### 4. 動態改變查詢條件（setWhere）

```javascript
// 依其他欄位的值動態限制 Refval 能查到的資料
$('#dfMaster_DEPT_CODE').combobox('setValue', 'HR');

// 部門改變時，重設下面的使用者 Refval 只能選 HR 的使用者
$('#dfMaster_USER_CODE').refval('setWhere', "DEPT_CODE = 'HR'");

// 清空使用者選擇（因為之前選的可能不是 HR 部門的人）
$('#dfMaster_USER_CODE').refval('setValue', '');
```

> **注意（原始碼 bug）**：`setWhere` 目前**只支援字串參數**（設定 `whereStr`）。傳 Array 的分支在 `bootstrap.infolight.js:10528-10530` 被註解掉了，實際上傳 Array 不會生效。要設多條件，用 SQL WHERE 字串拼接。

### 5. 在 DataForm 的 change 事件中連動

```javascript
// 綁定部門欄位的 change，連動更新使用者 Refval 的查詢範圍
$('#dfMaster_DEPT_CODE').on('change', function() {
    var dept = $(this).combobox('getValue');
    if (dept) {
        $('#dfMaster_USER_CODE').refval('setWhere', "DEPT_CODE = '" + dept + "'");
    } else {
        $('#dfMaster_USER_CODE').refval('setWhere', '1=1');  // 解除限制
    }
    $('#dfMaster_USER_CODE').refval('setValue', '');  // 清空避免殘留不合條件的值
});
```

### 6. 程式化開啟查詢彈窗

```javascript
// 直接開 Modal（通常是當 refval 綁在其他 UI 的按鈕上時使用）
$('#btnPickUser').click(function() {
    $('#dfMaster_USER_CODE').refval('openModal');
});

// 帶預設搜尋字開啟（模擬使用者在 input 打字後按 Enter）
$('#dfMaster_USER_CODE').data('search', 'J');   // 查詢前綴為 J 的值
$('#dfMaster_USER_CODE').refval('openModal');
```

### 7. DataGrid 內編輯的 Refval 存取

DataGrid 編輯模式下，Refval 包在 cell 的 editor 物件裡，不能直接 `$('#id')`：

```javascript
// 取得正在編輯那一列的 Refval 值
var $grid = $('#dgMain');
var editIndex = $grid.datagrid('options').editIndex;
var trs = $grid.datagrid('getTrs', editIndex);

// 從 td 的 data-field 定位欄位
var editor = trs.find('td[data-field="USER_CODE"]').data('editor');
if (editor && editor.type === 'refval') {
    // 取值
    var val = $.fn.datagrid.defaults.editors.refval.getValue(editor.target);

    // 設值
    $.fn.datagrid.defaults.editors.refval.setValue(editor.target, 'U002');

    // 或直接呼叫 refval widget 的 API
    $(editor.target).refval('setValue', 'U002');
}
```

或使用 DataGrid 的通用 editor API：

```javascript
// 方便的寫法
$('#dgMain').datagrid('setEditorValue', { field: 'USER_CODE', value: 'U002' });
var val = $('#dgMain').datagrid('getEditorValue', 'USER_CODE');
```

### 8. columnMatchs 自動回寫的運用

若設計時設定了 `columnMatchs`（TargetField ← SourceField），選取時會自動把選中列的其他欄位值寫入：

```
使用者 Refval 的 columnMatchs:
  DEPT_CODE <- DEPT_CODE   // 部門自動帶入
  USER_NAME <- USER_NAME   // 姓名自動帶入
```

選取使用者後，`DEPT_CODE`、`USER_NAME` 欄位會自動被填值且觸發 `$.triggerValueChanged`。**若 target 欄位本身是 refval，會連鎖觸發 blur（自動查顯示文字）**。

若要手動觸發回寫（例如在其他事件中強制重跑）：

```javascript
var opts = $('#dfMaster_USER_CODE').refval('options');
var val = $('#dfMaster_USER_CODE').refval('getValue');

$.getDisplayRow(opts, val, function(row) {
    if (row) {
        $('#dfMaster_USER_CODE').refval('doColumnMatch', row);
    }
}, true);
```

### 9. 驗證：失焦時自動檢查值存在

`checkData` 屬性為 true（預設）時，失焦會驗證：

```javascript
// 手動觸發同樣的驗證流程（例如 submit 前）
$('#dfMaster_USER_CODE').trigger('blur');

// 若不希望使用者手輸，設 selectOnly = true（input 會變 readonly，只能從面板選）
```

### 10. 配合 getWhereItems 取得完整查詢條件

`getWhereItems` 會組合「設計時 whereItems」＋「目前 search 關鍵字」：

```javascript
// 取得此 Refval 打開 Modal 時會傳給 datagrid 的完整 whereItems
var wheres = $('#dfMaster_USER_CODE').refval('getWhereItems');
console.log(wheres);
// [
//   { field: 'USER_CODE', operator: '%', value: '...' },  // 使用者在 input 按 Enter 的搜尋字
//   ...設計時 whereItems 內設定的條件（Constant / Variable / Row / Parent 等）
// ]
```

### 常見陷阱

| 陷阱 | 說明 | 解法 |
|------|------|------|
| `setValue` 後立刻讀 text 拿到空 | 內部有 100ms delay + async 載入 | 用 `$.getDisplayRow` 同步查詢 |
| `setWhere` 傳 Array 沒效果 | 原始碼 L10528 分支被註解 | 只傳字串 |
| DataGrid 內的 refval 找不到 | 無獨立 id，是 editor 物件 | 用 `trs.find('td').data('editor')` |
| onSelect 清空時 row 是 `{}` 而非 null | 邏輯判斷要用 `row.valueField` 才對 | `if (row && row.USER_CODE)` |
| 動態 `setWhere` 後舊值仍殘留 | whereStr 改了不會自動重查 | 手動 `setValue('')` 清空 |
| `setValue('')` 不會觸發 onSelect | 只有 blur/select 才會觸發 | 手動呼叫 onSelect 或 trigger blur |

## 備註

- `WhereItem` 是共用類別，Combobox、Options、Autocomplete、Submenu 都引用 `Refval.WhereItem`。
- `ColumnMatch` 可將選取列的其他欄位回寫到父容器的對應欄位。
- Column.Format 支援多種格式化：N0（數字）、日期、邏輯值、圖片、檔案、條碼、QRCode、地圖、簽名、鑽取、流程旗標等。
