# DataForm

> `EEPRWDTools.Core/Controls/DataForm.cs` — 564 行
> 繼承：`RWDContainerControl` → `RWDControl` → `Component`

## 用途

**資料表單元件**（Data Form）。

DataForm 是 EEP Core RWD 前端最主要的表單元件，用於編輯主檔（Master）記錄。設計師在模組 JSON 中以 `"type": "dataform"` 宣告，指定 RemoteName、欄位定義、工具列按鈕、查詢面板等設定。Runtime 時會根據 `Mode` 屬性決定渲染為 Modal Dialog、Panel 或 Tab 形式，並自動產生 Bootstrap 表單 HTML。

DataForm 支援：
- 多欄水平排版（`HorizontalColumnsCount`）
- 工具列按鈕（ToolItems）
- 內嵌查詢面板（QueryColumns）
- 預設值（Default）、驗證（Validate）、自動編號（AutoSeq）整合
- 欄位安全控管（Hidden）
- 多國語系（`[Localization]`）

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **RemoteName** | string | 命令選擇器 `[DataOption, CommandEditor]` | — | 綁定的 InfoCommand id |
| **Columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | 空 | 表單欄位定義集合 |
| **WhereStr** | string | 文字 `[DataOption]` | — | 預設 WHERE 條件字串 |
| **HorizontalColumnsCount** | int | 數字框 `[NumberboxEditor(1, true, 1)]` | 1 | 水平欄位數（表單分幾欄排列） |
| **Mode** | FormMode | 下拉 `[DataOption]` | — | 表單模式：`Dialog`（彈出視窗）/ `Panel`（面板）/ `Tab`（頁籤） |
| **Fit** | bool | 勾選 | false | Dialog 模式下是否撐滿螢幕 |
| **FormCls** | string | 文字（field） | — | 額外的 CSS class |
| **ToolItems** | List\<ToolItem\> | 集合編輯器 `[CollectionEditor]` | 空 | 工具列按鈕定義 |
| **ToolItemPosition** | ToolItemPosition | 下拉 | Top | 工具列位置：`Top` / `Bottom` |
| **Title** | string | 文字 `[DataOption, Localization]` | — | 表單標題（支援多國語系） |
| **QueryColumns** | List\<QueryColumn\> | 集合編輯器 `[DataOption(false), CollectionEditor]` | 空 | 查詢面板欄位（使用 DataGrid.QueryColumn） |
| **QueryMode** | QueryMode | 下拉 | — | 查詢模式：`Dialog` / `Panel` / `Fuzzy` |
| **QueryTitle** | string | 文字 `[Localization]` | — | 查詢面板標題 |
| **QueryColumnsCount** | int | 數字框 `[NumberboxEditor(1, true, 2)]` | 2 | 查詢面板水平欄位數 |
| **DuplicateCheck** | bool | 勾選 `[DataOption]` | false | 是否啟用重複資料檢查 |
| **ValidateStyle** | ValidateStyle | 下拉 `[DataOption]` | — | 驗證提示方式：`Hint`（提示）/ `Dialog`（對話框）/ `Timely`（即時） |
| **CloseProtect** | bool | 勾選 `[DataOption]` | false | 關閉時是否提醒未儲存變更 |
| **Autocomplete** | bool | 勾選 `[DataOption]` | false | 表單是否啟用瀏覽器自動完成 |
| **IsShowFlowIcon** | bool | 勾選 `[DataOption]` | false | 是否顯示流程圖示 |
| **AutoPause** | bool | 勾選 `[DataOption]` | false | 是否自動暫停 |
| **PId** | string | — `[DataOption(false)]` | — | 父元件 id（由容器自動設定） |

## 工具列按鈕（ToolItems）

`GetToolItems()` 提供的預設按鈕定義：

| Text | IconCls | Onclick | 說明 |
|------|---------|---------|------|
| Add | glyphicon-plus | `insert_row` | 新增記錄 |
| Edit | glyphicon-edit | `edit_row` | 編輯記錄 |
| Delete | glyphicon-remove | `delete_row` | 刪除記錄 |
| Save | glyphicon-floppy-disk | `submit` | 儲存變更 |
| Cancel | glyphicon-remove | `reload` | 取消 / 重新載入 |

設計師可在 JSON 的 `toolItems` 陣列中自訂按鈕，每個按鈕支援 `helpText`（popover 說明）、`btnCls`（自訂按鈕樣式）、`iconAlign`（圖示左右位置）及 `hidden`（隱藏）。

## 事件

| 事件 | 參數 | 預設回傳 | 說明 |
|------|------|----------|------|
| **OnLoad** | `row` | — | 表單載入資料後觸發 |
| **OnDelete** | `row` | `return true;` | 刪除前觸發，回傳 false 可取消刪除 |
| **OnApply** | （無） | `return true;` | 送出前觸發，回傳 false 可取消送出 |
| **OnApplied** | `data` | — | 送出成功後觸發 |
| **OnApplyError** | `message` | — | 送出失敗時觸發 |
| **OnCancel** | （無） | — | 取消操作時觸發 |
| **OnShowTitle** | `title, status, row` | — | 標題顯示時觸發，可動態修改標題 |

所有事件均使用 `[ScriptEditor]` 於設計介面編輯 JavaScript。

## 內部類別

### Column

表單欄位定義（繼承 `RWDCollectionItem`，實作 `IFormColumn`）。

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| Title | string | 文字 `[DataOption, Localization]` | — | 欄位標題（多國語系） |
| Field | string | 欄位選擇器 `[DataOption, ColumnEditor]` | — | 綁定的資料欄位名稱 |
| NewRow | bool | 勾選 | false | 是否強制換行 |
| Span | int | 數字框 `[NumberboxEditor(1, true, 1)]` | 1 | 佔用欄數（相對於 HorizontalColumnsCount） |
| Hidden | bool | 勾選 `[Security]` | false | 是否為隱藏欄位（渲染為 `<input type="hidden">`） |
| Editor | RWDEditor | 編輯器選擇器 `[EditorOptionEditor]` | Textbox | 欄位編輯器元件（Textbox、Combobox 等） |

### ToolItem

工具列按鈕定義（實作 `IToolItem`）。

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| Text | string | 文字 `[Localization]` | 按鈕文字（多國語系） |
| HelpText | string | 文字 `[Localization, MergeKey]` | Popover 說明文字 |
| IconCls | string | 圖示選擇器 `[IconRWDEditor]` | 圖示 CSS class（glyphicon 或 fa） |
| IconAlign | IconAlign | 下拉 | 圖示位置：`Left` / `Right` |
| BtnCls | string | 按鈕樣式選擇器 `[BtnClsEditor]` | 按鈕 CSS class（空值時使用 `dataform-btn`） |
| Hidden | bool | 勾選 | 是否隱藏 |
| Onclick | string | 文字 `[DataOption]` | 點擊事件（如 `insert_row`、`submit`） |

### QueryForm（衍生類別）

繼承 DataForm 的查詢表單。由 DataForm 內部自動建立，當 `QueryColumns` 有定義時渲染於表單上方。覆寫行為：
- `IsDialog` — 依 `QueryMode == Dialog` 決定
- `IsFuzzyQuery` — 依 `QueryMode == Fuzzy` 決定
- `RenderButtons()` — 產生 Query / Close（或 Clear）按鈕

### LogonForm（衍生類別）

登入表單。固定為 Dialog 模式，內建 User 和 Password 兩個欄位，按鈕為 Logon / Cancel。

### PromtForm（衍生類別）

提示表單。按鈕為 OK / Cancel，無查詢功能。

## 前端使用範例

以下範例展示 DataForm 常用事件的 JavaScript 寫法。這些腳本透過設計介面的 `[ScriptEditor]` 編輯，或直接寫在模組 JSON 的對應事件屬性中。

### OnLoad — 表單載入後處理

當表單載入一筆資料時觸發，可用於動態控制欄位狀態。

```javascript
// 參數：row（當前載入的資料列）
// 根據狀態欄位動態停用編輯
if (row.STATUS === 'C') {
    // 已結案的記錄，停用所有輸入欄位
    this.find('[name="USERID"]').textbox('disable');
    this.find('[name="USERNAME"]').textbox('disable');
}

// 根據資料動態設定欄位值
if (!row.CREATE_DATE) {
    this.find('[name="CREATE_DATE"]').datebox('setValue', new Date().format('yyyy/MM/dd'));
}
```

### OnApply — 送出前驗證

送出前觸發，回傳 `false` 可取消送出。適合做自訂驗證邏輯。

```javascript
// 參數：（無）
// 回傳：return true; （預設，允許送出）

var userId = this.find('[name="USERID"]').textbox('getValue');
var userName = this.find('[name="USERNAME"]').textbox('getValue');

// 自訂驗證：代號必須為英數字
if (!/^[A-Za-z0-9]+$/.test(userId)) {
    alert('使用者代號僅允許英數字');
    return false;
}

// 確認提示
if (!confirm('確定要儲存嗎？')) {
    return false;
}

return true;
```

### OnApplied — 送出成功後處理

送出成功後觸發，可用於重新整理關聯元件或顯示提示。

```javascript
// 參數：data（伺服器回傳的資料）

// 重新載入關聯的 DataGrid
var grid = page.find('#detailGrid');
if (grid.length > 0) {
    grid.datagrid('reload');
}

// 顯示成功訊息
$.messager.show({
    title: '提示',
    msg: '資料儲存成功',
    timeout: 2000
});
```

### OnApplyError — 送出失敗處理

送出失敗時觸發，可用於自訂錯誤處理。

```javascript
// 參數：message（伺服器回傳的錯誤訊息）

// 自訂錯誤提示
$.messager.alert('儲存失敗', '資料儲存發生錯誤：' + message, 'error');

// 記錄到 console 便於除錯
console.error('[DataForm] Apply error:', message);
```

### OnCancel — 取消操作處理

使用者按下 Cancel 按鈕時觸發。

```javascript
// 參數：（無）

// 關閉前確認
if (!confirm('尚有未儲存的變更，確定要取消嗎？')) {
    return;
}

// 清空自訂的暫存資料
sessionStorage.removeItem('tempFormData');
```

### OnShowTitle — 動態修改標題

標題顯示時觸發，可根據操作狀態動態調整標題文字。

```javascript
// 參數：title（原始標題）, status（目前狀態：insert/edit/view）, row（資料列）

if (status === 'insert') {
    return title + ' - 新增';
} else if (status === 'edit') {
    return title + ' - 編輯 [' + row.USERID + ']';
}
return title;
```

## JavaScript API 操作範例

前面「前端使用範例」章節介紹的是事件 hook 的寫法。本節示範**從外部或其他事件主動呼叫 DataForm API** 的實戰 pattern，例如在按鈕點擊、父表選列、或另一個 Tab 的 postMessage 中操作 DataForm。

### 1. 取值 / 設值

```javascript
// 取目前整筆 row（從表單各欄位收集）
var row = $('#dfMaster').form('getRow');
console.log(row.USER_CODE, row.USER_NAME);

// 排除某些型別欄位（例如排除 refval）
var row = $('#dfMaster').form('getRow', ['refval', 'fileupload']);

// 取/設單一欄位（依欄位類型使用對應 API）
$('#dfMaster_USER_CODE').textbox('getValue');
$('#dfMaster_USER_NAME').textbox('setValue', 'Alice');
$('#dfMaster_DEPT').combobox('setValue', 'HR');
$('#dfMaster_BIRTH').datebox('setValue', '2026/04/17');

// 整個 row 回填（會呼叫所有欄位的 setValue）
$('#dfMaster').form('loadRow', {
    USER_CODE: 'U001',
    USER_NAME: 'Alice',
    DEPT: 'HR',
    BIRTH: '2026/04/17'
});

// 清空整個表單
$('#dfMaster').form('clear');
```

### 2. 程式化觸發 CRUD 操作

```javascript
// 新增
$('#dfMaster').form('insert_row');    // 觸發 onLoad（empty row），設 status='inserted'

// 編輯
$('#dfMaster').form('edit_row');      // 從 ViewGrid 選列載入，設 status='updated'

// 送出儲存
$('#dfMaster').form('submit');

// 刪除（含確認對話框 + 送後端）
$('#dfMaster').form('delete_row');

// 取消（等同 reload 當前頁）
$('#dfMaster').form('cancel');

// 關閉表單（Panel→closeCurrentTab / Dialog→modal hide）
$('#dfMaster').form('close');
```

### 3. 動態切換狀態

```javascript
// 取得當前狀態
var st = $('#dfMaster').form('status');   // 'view' | 'inserted' | 'updated'

// 設定狀態（含欄位 readonly 切換、按鈕顯隱、key 欄位保護）
$('#dfMaster').form('status', 'view');     // 轉唯讀檢視
$('#dfMaster').form('status', 'updated');  // 轉編輯（key 欄位仍鎖住）
$('#dfMaster').form('status', 'inserted'); // 轉新增（key 欄位可編）
```

**何時用**：例如在按下特殊按鈕（非 Add/Edit）進入自訂模式時手動切狀態。

### 4. 程式化驗證 / 清錯誤樣式

```javascript
// 跑一次完整驗證（XSS → required → validType），回傳 bool
var valid = $('#dfMaster').form('validate');
if (!valid) {
    // validateStyle 若為 dialog 會自動跳 alert；timely 會在欄位下顯示紅字
    return;
}

// 清除所有欄位的 has-error 樣式與錯誤訊息
$('#dfMaster').form('resetValidate');
```

### 5. 動態查詢條件（setWhere + reload）

```javascript
// Array 形式（每筆 whereItem）
$('#dfMaster').form('setWhere', [
    { field: 'DEPT', operator: '=', value: 'HR' },
    { field: 'STATUS', operator: '<>', value: 'C' }
]);

// String 形式（原生 SQL WHERE）
$('#dfMaster').form('setWhere', "DEPT = 'HR' AND STATUS <> 'C'");

// setWhere 內部會呼叫 reload(1) 自動重查
```

### 6. 取得關聯物件（重要！）

DataForm 與查詢面板、檢視元件、圖表是**鬆耦合**關係（透過 `queryObj` / `editForm` 互相對應）。要操作它們有專用方法：

```javascript
// ViewGrid — 以此 form 為 editForm 的 DataGrid（主從關係中的「從」就是本表單）
var $viewGrid = $('#dfMaster').form('getViewGrid');
$viewGrid.datagrid('reload');

// ViewObj — 關聯的檢視元件（DataGrid / DataList / Schedule / Tree / Orgchart 擇一）
var $view = $('#dfMaster').form('getViewObj');

// 重新載入所有關聯檢視（一次刷新 Grid + Chart + Gantt 等）
$('#dfMaster').form('reloadViewObj');

// DetailGrid — 以此 form 為 parentObject 的明細 DataGrid
var $detail = $('#dfMaster').form('getDetailGrid');
$detail.datagrid('options').pagination;

// 取得所有子層（含孫層）明細
var $allDetails = $('#dfMaster').form('getDetailGrid', true);  // recursive=true

// 查詢面板（本 form 作為查詢面板時，對應的目標物件）
var $qGrid   = $('#dfQuery').form('getQueryGrid');    // 被此查詢面板查詢的 DataGrid
var $qForm   = $('#dfQuery').form('getQueryForm');    // 對應的 DataForm
var $qReport = $('#dfQuery').form('getQueryReport');  // ReportViewer

// 圖表類關聯（通常由查詢面板觸發）
$('#dfQuery').form('getBarChart');
$('#dfQuery').form('getLineChart');
$('#dfQuery').form('getPieChart');
$('#dfQuery').form('getDonutChart');
$('#dfQuery').form('getGantt');
$('#dfQuery').form('getPivotTable');
```

### 7. 組合範例：主表存檔後刷新明細

```javascript
// 在 onApplied 事件中
function onApplied(data) {
    // 重新載入所有關聯檢視（ViewGrid + 圖表）
    $(this).form('reloadViewObj');

    // 或只刷新明細
    $(this).form('getDetailGrid').datagrid('reload');

    // 或通知父 Tab（Tab 模式）
    $(this).form('refreshTabGrid');
}
```

### 8. 組合範例：submit 前自訂驗證明細

```javascript
// 在 onApply 事件中
function onApply() {
    // 內建 validate 會做主表 + 子表（含 endEdit）驗證
    // 但若要額外業務檢核（例如：明細至少 1 筆、總金額 > 0）

    var $detail = $(this).form('getDetailGrid');
    var rows = $detail.datagrid('getRows');

    if (rows.length === 0) {
        $.alert('請至少新增 1 筆明細', 'warning');
        return false;
    }

    var total = 0;
    for (var i = 0; i < rows.length; i++) total += parseFloat(rows[i].AMOUNT) || 0;
    if (total <= 0) {
        $.alert('總金額必須大於 0', 'warning');
        return false;
    }

    return true;
}
```

### 9. 組合範例：動態隱藏/停用某欄位

```javascript
// 在 onLoad 事件中依狀態決定欄位可編
function onLoad(row) {
    // 收單後的記錄：明細欄位都鎖
    if (row.STATUS === 'CLOSED') {
        $(this).find('[name="AMOUNT"]').textbox('readonly', true);
        $(this).find('[name="REMARK"]').textbox('readonly', true);
    }

    // 依角色隱藏欄位
    var clientInfo = JSON.parse(sessionStorage.clientInfo);
    if (!clientInfo.groups.includes('HR')) {
        $(this).find('[name="SALARY"]').closest('.form-group').hide();
    }
}
```

### 10. 組合範例：匯出表單當前資料為 Word

```javascript
// 工具列自訂按鈕上
function onClickExport() {
    var form = this;  // DataForm jQuery 物件

    // 依當前 row 產生 Word
    form.form('exportWord');                  // .docx
    form.form('exportWordPdf');               // .pdf
    form.form('exportWordExcel', '表單範本'); // .xlsx
}
```

### 11. Panel 模式分頁切換

```javascript
// 從第 1 頁開始
$('#pfMaster').form('reload', 1);

// 直接跳第 5 頁
$('#pfMaster').form('reload', 5);

// 重設分頁顯示
$('#pfMaster').form('refreshPagination', 100);  // 告知總筆數 100
```

### 12. iframe / Tab 跨頁通知

```javascript
// 子 Tab 存檔後通知主 Tab 刷新對應 Grid（走 postMessage）
$('#dfMaster').form('refreshTabGrid');

// 自訂 postMessage 通知（需雙方約定格式）
window.top.postMessage({
    type: 'custom-event',
    formId: 'dfMaster',
    data: $('#dfMaster').form('getRow')
}, '*');
```

### 13. `form('open', ...)` 完整參數（Dialog 模式啟動）

> 來源：討論區 [#482483](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482483)

在 DataGrid ToolItem 或其他位置要「程式化」開啟一個 `Mode=Dialog` 的 DataForm 時，必須帶齊必要參數，否則會比原生按鈕缺欄位：

```javascript
// 直接開到新增狀態
$('#dfMaster').form('open', {
    row:    { '編號': '自動編號' },  // 初始值（新增時的預填）
    status: 'inserted',             // view / inserted / updated
    keys:   '編號'                  // Key 欄位名稱（updated 時需鎖住）
});

// 既有資料 → 編輯狀態（兩階段：open + setWhere 等資料載入、再改 status）
$('#DataForm1').form('open', {
    row: { 編號: '' },
    keys: '編號'
});
$('#DataForm1').form('setWhere', "編號='" + value + "'");
setTimeout(function () {
    $('#DataForm1').form('status', 'updated');
}, 300);  // 等 setWhere 的 reload 完成
```

`open` 完整接受的屬性：

| 屬性 | 類型 | 說明 |
|------|------|------|
| `row` | Object | 預填資料列 |
| `status` | string | `view` / `inserted` / `updated` |
| `keys` | string | Key 欄位名稱（逗號分隔） |
| `parentRow` | Object | 主表的一列（明細表在 Dialog 模式下需要帶） |
| `onLoadDetail` | Function | 明細載入後的 callback |

### 14. 明細 DataGrid 加總自動同步到主表 DataForm

> 來源：討論區 [#482366](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482366) / [#482217](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482217)

**情境**：明細 Grid 的某欄異動 → 主表 DataForm 的加總欄位要即時更新。

**解法**：不需要寫 JS 手動加總。只要：

1. 明細 DataGrid 該欄位的 **`Total` 屬性設為 `sum`**
2. 主表 DataForm 有**同名欄位**
3. 系統會自動把明細的 Footer 合計寫到主表對應欄位

```javascript
// 明細欄位設定（JSON）
{
    "field": "AMOUNT",
    "title": "金額",
    "total": "sum",   // ← 關鍵
    "format": "N0"
}
```

### 15. 自訂 DataForm 用 modal('show') 顯示

> 來源：討論區 [#481667](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481667)

若設計介面上的 DataForm 不是由 DataGrid 觸發、而是要**自己在別處開啟**，可以直接用 Bootstrap modal API：

```javascript
// 顯示自訂的 DataForm modal
$('#AllSetup').modal('show');
$('#AllSetup').modal('hide');

// 開啟後以 setWhere 指定要顯示哪一筆（預設進 view 狀態）
$('#AllSetup').form('setWhere', "員工編號 = '002'");

// 若要直接進編輯：setWhere → setTimeout → status('updated')（見第 13 點）
```

**與 `form('open')` 的差別**：

- `modal('show')` — 純 Bootstrap 顯示命令，不會觸發 DataForm 的初始化流程
- `form('open')` — 會完整觸發 `loadRow` / `loadDetail` / `status` / `onShowTitle` / `onLoad` / `onFlowLoad` 等事件

想要完整觸發事件，用 `form('open')`；只是要顯示已由其他機制載好資料的 form，用 `modal('show')`。

### 常見陷阱

| 陷阱 | 說明 | 解法 |
|------|------|------|
| **`getRow` 拿不到某欄位值** | 該欄位元件類型未註冊在 form 內部的 collector | 用 `$('#dfMaster_XXX').xxx('getValue')` 直接拿 |
| **`setWhere` 後沒刷新** | `setWhere` 內部會 reload，若又手動 reload 會送兩次 | 不要再手動呼叫 reload |
| **`setWhere` 是非同步，接著 `edit_row` 用到舊資料** | `setWhere` 觸發的 reload 還沒完成就 edit_row | 用 `setTimeout` 延遲 300ms 左右等資料載入。來源：[#482053](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482053) |
| **`onApplied` 內 `return false` 無效 / 導致異常** | `onApplied` 時機點框架不接收回傳值 | 不要在 `onApplied` 寫 return。來源：[#482366](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482366) |
| **`status('view')` 後 required 紅字沒消失** | 狀態切換會刷新 label 樣式但 refval 特殊處理 | 檢查 refval 是否有 `columnMatchs` 導致 blur 連鎖觸發 |
| **`insert_row` 後欄位預設值沒填入** | 需要 Default 元件同 Container 且 `BindingObject = form.Id` | 確認 Default 元件的 `bindingObject` 屬性 |
| **`delete_row` 按下沒反應** | `onDelete` 或 `onFlowDelete` 回傳 false 中斷 | 檢查事件程式 |
| **關聯 Grid 找不到** | `getViewGrid` 依 `editForm = {本 form id}` 比對 | 檢查 DataGrid 的 `editForm` 屬性 |
| **Key 欄位編輯時仍可改** | `status` 為 `inserted` 時 key 可編、`updated` 才鎖 | 用 edit_row 觸發就會正確鎖定 |
| **Tab 模式 close 後回到原頁面沒更新** | 沒呼叫 `refreshTabGrid` | 存檔 / 刪除後呼叫它 |
| **DataPanel 欄位隱藏時連帶前一欄一起消失** | `.parent().parent().hide()` 多抓了一層 | 一層 `.parent().hide()` 就夠。來源：[#482509](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482509) |
| **自訂 ToolItem 按鈕只在 view 狀態顯示** | 修改狀態下系統會隱藏非預期的 ToolItem | 需改底層 `refreshPagination` 加 OR 條件（升級會被覆蓋，需記錄）。來源：[#481937](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481937) / [#481830](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481830) |

## 架構建議：主明細 Key 關聯用 `infoDataSource`（不要用 onblur submit）

> 來源：討論區 [#482440](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482440)

常見需求：主表 DataForm 有自動編號欄位，明細表的 KEY 欄位要自動帶入主表編號。

**不建議的做法（前端硬串）**：
```javascript
// ❌ 在主表欄位 onblur 觸發 submit、讀 Identity、再 reload
$('#dfMaster_編號').on('blur', function() {
    $('#dfMaster').form('submit');
    // 再想辦法撈 Identity、reload...
});
```

問題：主從存檔邏輯混亂、明細被迫 autoApply、錯誤處理複雜。

**建議的做法（server 端 infoDataSource）**：

1. 在 server 端設計 `infoDataSource` 設定主明細關聯
2. 關聯欄位用 `ParentValues` 自動帶入
3. 存檔時系統會：主表先存 → 取回 Identity → 自動寫入明細的關聯欄位 → 明細一併存

明細的 autoApply 保持 false，**整個主從作為一個 Transaction**。

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` — `$.fn.form = $.createObj('form', { ... })` (約 line 6345-8082)

### 初始化（init）

`init` 方法在元件建立時執行，負責以下工作：

1. **解析選項**：讀取 DOM 上的 `data-options` 屬性（`$.parseOptions`），存入 `jq.data('form').options`。
2. **Fuzzy 模式處理**：若 `mode == 'fuzzy'`，掃描所有查詢欄位的 `title`，組合成 placeholder 文字顯示於搜尋框。
3. **工具列按鈕本地化**：遍歷 `div.dataform-toolitem .btn`，將按鈕文字轉為 `$.fn.locale` 對應語系（`delete` → `remove`、`export` → `exports` 等特殊映射）。
4. **Fieldset 摺疊**：為每個 `<legend>` 前置一個 `glyphicon-minus` 圖示，點擊可收合/展開所屬的 `<form>` 區塊（`slideUp` / `slideDown`）。
5. **綁定事件**：呼叫 `bindEvent` 註冊所有交互事件。
6. **Panel 模式自動載入**：當 `mode == 'panel'` 且無關聯的 ViewObj 時，設定 `pagination = true` 並呼叫 `reload(1)` 載入第一頁資料。若有 drill-down 參數（`targetRemoteName`），則改用 `whereItems` 過濾。
7. **流程整合**：若 `isShowFlowIcon` 為 true，隱藏 submit/reload 按鈕，並依流程狀態（`isOpen` / `isPrepare`）呼叫對應的 `$.fn.flow` 方法。
8. **ChatUX 整合**：解析 URL 中的 `c=` 參數，從 `sessionStorage` 取得聊天指令（新增/更改/刪除/印表），自動設定 `loadAction` 和 `whereItems`。

### 事件綁定（bindEvent）

`bindEvent` 註冊以下互動行為：

| 綁定對象 | 事件 | 行為 |
|----------|------|------|
| `.form-submit` / `.form-close` / `.form-query` / `.form-clear` | click | 呼叫對應的 `form('submit')` / `form('close')` / `form('query')` / `form('clear')` 方法；clear 同時載入預設值 |
| `div.dataform-toolitem li`（分頁） | click | 依 `value`（first/previous/next/last/數字）計算頁碼，呼叫 `reload(page)` |
| `div.dataform-toolitem .btn`（工具列按鈕） | click | 讀取按鈕的 `onclick` 選項，若為已註冊方法則呼叫 `form(method)`，否則以 `$.callFunction` 執行自訂函式 |
| `.form-control[name]` | change | 觸發 `$.triggerValueChanged(control, name)` 通知 ValueChanged 事件 |
| `.form-control[name]`（validateStyle == 'timely'） | blur | 即時驗證：檢查 required 及 validType，錯誤時加上 `has-error` class 並顯示紅色錯誤訊息 |
| `shown.bs.modal` | modal shown | 計算 `modal-body` 的 `max-height`（視窗高度扣除 header/footer/margin），設定 `overflow-y` 捲動 |
| `hide.bs.modal` | modal hide | 若 `closeProtect` 啟用且狀態非 view，彈出確認對話框；使用者確認後設定 `forceClose` flag 才真正關閉 |
| `hidden.bs.modal` | modal hidden | 清除 `forceClose` flag，呼叫 `removeLock` 釋放記錄鎖定；若為流程開啟模式則關閉當前頁籤 |

### 資料載入（reload / loadRow / loadDetail）

**reload(page)**：

1. 顯示 loading 遮罩。
2. 透過 `$.loadData(remoteName, { rows: 1, page, whereStr, whereItems })` 向後端取得單筆資料。
3. 呼叫 `loadRow(row)` 填入表單、`loadDetail(row)` 載入明細 Grid、`status('view')` 設為檢視模式。
4. 隱藏 footer 按鈕，重新渲染分頁控制列（`refreshPagination`）。
5. 若有 `loadAction`（如 ChatUX 觸發的 `edit_row`），自動執行該動作。
6. 觸發 `onLoad(row)` 事件。

**loadRow(row)**：遍歷所有 `.form-control`，以 `name` 為 key 從 `row` 取值呼叫 `setValue()`。另處理 `creator`/`updater` 日期欄位及 `daterange` 元件的特殊初始化。

**loadDetail(data)**：取得所有 `parentObject` 指向本表單的明細 DataGrid，呼叫其 `datagrid('init', detailOptions)` 重新載入明細資料。`detailOptions` 包含 `parentTable`、`parentRow`、`onLoadDetail` 等。

### 清除（clear）

清空表單所有 `input` 和 `select` 的值。radio/checkbox 則取消勾選。`options-div` 內的 select 以 `prop('checked', false)` 處理。

### 開啟表單（open）

依 `mode` 決定開啟方式：

- **Tab 模式**：將表單資料序列化至 `sessionStorage`，透過 `window.top.addTab(mid, title, url)` 在主框架開啟新頁籤。頁籤 id 以 key 值的 MD5 產生，確保同筆資料不重複開啟。支援 `onShowTitle` 動態修改頁籤標題。
- **Dialog/Panel 模式**：呼叫 `$.checkSession` 確認 session 有效後，依序執行 `loadRow` → `loadDetail` → `status(data.status)` → `modal('show')`。觸發 `onShowTitle`、`onLoad`、`onFlowLoad` 事件。

### 送出（submit）

完整的送出流程：

```
submit()
  ├─ 若 status == 'view' → 直接關閉 Modal，不送出
  ├─ 呼叫 onApply 事件 → 回傳 false 可中斷
  ├─ 等待所有 fileupload 元件完成上傳（$.when）
  ├─ validate() 表單驗證
  ├─ 明細 Grid 驗證（endEdit + validate）
  ├─ 組裝 datas 陣列：
  │   ├─ 主表 { table, inserted:[], updated:[], deleted:[] }
  │   └─ 明細 Grid 的 getChangedDatas()
  ├─ autoApply == false 時：
  │   └─ 僅更新前端 ViewGrid 的 row，不送後端
  ├─ autoApply == true 時：
  │   ├─ $.updateData(remoteName, datas, duplicateCheck, ...)
  │   ├─ 成功：更新 ViewGrid、acceptChanges、removeLock、reloadViewObj、
  │   │        status('view')、觸發 onApplied / onFlowApplied
  │   └─ 失敗：duplicate 錯誤顯示重複提示、其他錯誤觸發 onApplyError / onFlowError
  └─ complete：重設 submit 按鈕狀態
```

### 驗證流程（validate）

1. 先呼叫 `resetValidate()` 清除先前的錯誤樣式。
2. 遍歷所有 `.form-control[name]`：
   - 先做 **XSS 驗證**（`$.xssValidate`）。
   - 再檢查 **required**：值為空且 `required:true` 時產生「必填」錯誤。
   - 再執行 **validType** 驗證（`$.validate(validType, value, field, jq)`）。
   - 錯誤時加上 `has-error` class。
3. 依 `validateStyle` 顯示錯誤：
   - `dialog`：以 `$.alert` 彈出所有錯誤訊息。
   - `hint`：在 `modal-body` / `panel-body` 前方插入 `alert-danger` 橫幅。
   - `timely`：在欄位下方即時顯示紅色錯誤文字（於 blur 事件中處理）。

### 欄位狀態控制（status）

`status(value)` 方法同時扮演 getter 和 setter。設定狀態時：

| 狀態 | 欄位行為 | 工具列按鈕 | 明細 Grid |
|------|----------|------------|-----------|
| `view` | 所有欄位 `setReadonly(true)`；required 標記移除紅色 | 顯示 Add/Edit/Delete 等操作按鈕，隱藏 Save/Cancel | `datagrid('readonly', true)` |
| `inserted` | 依各欄位原始 `readonly` 設定還原；`creator` 元件自動填入日期；`refval` 元件若有 `columnMatchs` 則觸發 blur | 顯示 Save/Cancel，隱藏其他 | `datagrid('readonly', false)` |
| `updated` | **Key 欄位**強制 `setReadonly(true)`，其餘依原始設定還原 | 顯示 Save/Cancel，隱藏其他 | `datagrid('readonly', false)` |

非 view 狀態下，`required:true` 的欄位標題加上 `text-danger` 紅色樣式。

切換狀態時也會重新載入含有 `row` 參數的 `combobox`（動態篩選下拉選項）。

### 工具列動作（ToolItem Actions）

| 方法 | 行為 |
|------|------|
| `insert_row` | 取得預設值（`getDefaultValues`），載入空白列，狀態設為 `inserted`，重設明細 Grid |
| `edit_row` | 取得目前表單資料列，狀態設為 `updated`；觸發 `onFlowUpdate` 事件 |
| `delete_row` | 觸發 `onDelete` + `onFlowDelete` 事件（可取消）→ 確認對話框 → `$.updateData` 送出刪除 → `reload` 重新載入 → `reloadViewObj` 刷新關聯元件 → 觸發 `onApplied` |
| `submit` | 見上方「送出」流程 |
| `reload` | 等同 `cancel`，呼叫 `reload()` 重新載入當前頁 |
| `close` | Panel 模式呼叫 `window.top.closeCurrentTab()`；Dialog 模式呼叫 `modal('hide')`；觸發 `onCancel` |
| `exportWord` / `exportWordPdf` / `exportWordExcel` | 收集表單資料，透過 `$.exportFile('word', ...)` 產生 Word/PDF/Excel 匯出檔案 |

### 查詢（query）

1. 遍歷查詢表單的所有 `.form-control`，組裝 `whereItems` 陣列（每個項目含 field、operator、value、dataType 等）。
2. Fuzzy 模式下所有條件加上 `or: true`。
3. 將 whereItems 傳送至關聯的 DataGrid（`setWhere`）、ReportViewer、PivotTable、Gantt、各類 Chart。
4. 若查詢面板為 Dialog 模式，查詢後自動關閉 Modal。

### 分頁（refreshPagination）

Panel 模式下每次 `reload` 後更新分頁列。資料筆數為 0 時隱藏 Edit/Delete 按鈕。分頁採前後各顯示 1 頁的滑動窗口演算法，含「首頁 / 上一頁 / 下一頁 / 末頁」按鈕。

### 流程整合（Workflow / Flow）

DataForm 透過 `isShowFlowIcon` 選項與 `$.fn.flow` 模組整合：

- **init 時**：呼叫 `$.fn.flow.showFlowIcons(target)` 顯示流程圖示按鈕；隱藏原生的 submit/reload 按鈕。
- **flowOpened**：流程已開啟（簽核中）時進入 flowOpened 模式。
- **flowPrepare**：流程待送簽時顯示流程操作按鈕。
- **submit 時**：觸發 `onFlowApply` → 送出成功後觸發 `onFlowApplied`；失敗時觸發 `onFlowError`。
- **hidden.bs.modal**：流程開啟模式下，Dialog 關閉後自動呼叫 `window.top.closeCurrentTab()` 關閉頁籤。
- **事件攔截**：`insert_row` 和 `edit_row` 會先觸發 `onFlowUpdate`，`delete_row` 會先觸發 `onFlowDelete`，回傳 false 可取消操作。

### 記錄鎖定（Record Lock）

`removeLock(row)` 方法在 Modal 關閉（`hidden.bs.modal`）及送出成功後呼叫。若關聯的 ViewGrid 啟用 `recordLock`，透過 `$.removeLock(remoteName, keys, [row])` 釋放伺服器端的記錄鎖。

### 公開 API 方法一覽

| 方法 | 參數 | 回傳 | 說明 |
|------|------|------|------|
| `options()` | — | Object | 取得元件選項 |
| `init(options)` | options | jq | 初始化 |
| `initQuery()` | — | jq | 初始化查詢面板（設定預設值、readonly、Enter 鍵攔截） |
| `reload(page)` | page: number | jq | 載入指定頁資料 |
| `open(data)` | { row, keys, status, parentRow, onLoadDetail } | jq | 開啟表單（Dialog/Tab） |
| `clear()` | — | jq | 清空所有欄位值 |
| `loadRow(row)` | row: Object | jq | 將 row 資料填入表單 |
| `loadDetail(data)` | { row, parentRow, onLoadDetail } | jq | 載入明細 Grid |
| `status(value)` | value?: string | jq \| string | 設定/取得狀態（view/inserted/updated） |
| `getRow(excludeTypes)` | excludeTypes?: array | Object | 從表單收集目前資料列 |
| `getDefaultValues(values)` | values?: Object | Object | 取得欄位預設值（含 Default 元件、carryOn、ChatUX） |
| `insert_row()` | — | jq | 新增記錄 |
| `edit_row()` | — | jq | 編輯目前記錄 |
| `delete_row()` | — | jq | 刪除目前記錄 |
| `submit()` | — | jq | 送出（儲存） |
| `cancel()` | — | jq | 取消（等同 reload） |
| `close()` | — | jq | 關閉表單 |
| `validate()` | — | bool | 執行驗證，回傳是否通過 |
| `resetValidate()` | — | jq | 清除驗證錯誤樣式 |
| `readonly(value)` | value: bool | jq | 隱藏/顯示 Modal 的 close 和 footer 按鈕 |
| `query()` | — | jq | 執行查詢，將條件傳至關聯元件 |
| `openQuery()` | — | jq | 開啟查詢面板 Dialog |
| `setWhere(where)` | where: Array \| string | jq | 設定篩選條件並 reload |
| `getViewObj()` | — | jq | 取得關聯的檢視元件（DataGrid/DataList/Schedule/Tree/Orgchart） |
| `getViewGrid()` | — | jq | 取得關聯的 DataGrid（editForm 指向本表單者） |
| `getDetailGrid(recursive)` | recursive?: bool | jq | 取得明細 Grid（parentObject 指向本表單者） |
| `reloadViewObj()` | — | jq | 重新載入所有關聯的檢視元件 |
| `refreshPagination(total)` | total: number | jq | 更新分頁控制列 |
| `exportWord(fileType)` | fileType?: string | jq | 匯出 Word |
| `exportWordPdf()` | — | jq | 匯出 PDF |
| `exportWordExcel(name)` | name?: string | jq | 匯出 Excel |
| `removeLock(row)` | row?: Object | jq | 釋放記錄鎖定 |
| `refreshTabGrid()` | — | jq | Tab 模式下透過 postMessage 通知主頁籤重新載入 Grid |
| `onEvent(param)` | { events, parameters } | bool | 依序觸發指定事件，任一回傳 false 則中斷 |

## 技術參考（AI / 深度使用）

> 此區段彙整結構化內容 — JSON schema 範例、伺服器端渲染管線 — 供 AI 檢索或需要深入理解元件行為的開發者參考。日常設定看前面的「設計介面屬性」、「事件」、「JavaScript API 操作範例」即可。

### JSON 設定範例

```json
{
  "type": "dataform",
  "id": "masterForm",
  "remoteName": "masterCmd",
  "mode": "Dialog",
  "title": "主檔編輯",
  "horizontalColumnsCount": 2,
  "duplicateCheck": true,
  "validateStyle": "Hint",
  "closeProtect": true,
  "columns": [
    { "field": "USERID", "title": "使用者代號", "span": 1 },
    { "field": "USERNAME", "title": "使用者名稱", "span": 1 },
    { "field": "EMAIL", "title": "電子郵件", "span": 2, "newRow": true, "editor": { "type": "textbox" } }
  ],
  "toolItems": [
    { "text": "Add", "iconCls": "glyphicon-plus", "onclick": "insert_row" },
    { "text": "Save", "iconCls": "glyphicon-floppy-disk", "onclick": "submit" }
  ]
}
```

### 渲染流程（伺服器端）

```
Render()
  ├─ RenderQuery()          → 若有 QueryColumns，建立 QueryForm 並渲染
  ├─ 判斷 IsDialog
  │   ├─ true  → 渲染 Bootstrap Modal（modal-dialog / modal-content）
  │   └─ false → 渲染 Bootstrap Panel（panel-group / panel-collapse）
  ├─ RenderToolItems()      → 依 ToolItemPosition 在表單上方或下方渲染工具列
  ├─ RenderForm()           → 渲染 <form> 及所有欄位
  │   ├─ 整合 Default / Validate / AutoSeq 元件設定
  │   ├─ VisibleColumns → 依 HorizontalColumnsCount 排版
  │   └─ HiddenColumns → 渲染為 hidden input
  ├─ RenderChildren()       → 渲染子控件（如 DataPanel）
  └─ RenderButtons()        → 渲染 OK / Cancel 按鈕（footer）
```

依 `Mode` 差異：`Dialog` / `Tab` 渲染為 Bootstrap Modal，`Panel` 渲染為可摺疊 Panel。`HorizontalColumnsCount` 控制欄位寬度（`colSpan = 12 / horizontalColumnsCount * span - labelSpan`）。

## 備註

- DataForm 的 `Mode` 決定渲染方式：`Dialog` 和 `Tab` 渲染為 Bootstrap Modal，`Panel` 渲染為可摺疊的 Panel。
- `HorizontalColumnsCount` 控制 Bootstrap Grid 的欄位寬度計算：`colSpan = 12 / horizontalColumnsCount * span - labelSpan`。當 `HorizontalColumnsCount >= 4` 時，label 佔 1 格；否則佔 2 格。
- 表單渲染時會自動整合同一容器內的 `Default`、`Validate`、`AutoSeq` 元件，將預設值和驗證規則注入到對應的 Column。
- `DuplicateCheck` 啟用時，前端會在送出前檢查是否有重複資料。
- `CloseProtect` 啟用時，使用者關閉視窗會收到未儲存變更提示。
- `FormCls` 為直接 field（非 property），可在程式碼中設定額外的 CSS class。
- ToolItem 的 `Render()` 方法會判斷 `IconCls` 前綴：以 `fa` 開頭使用 Font Awesome，否則使用 Glyphicon。
- DataPanel 子控件的 `PId` 會在 `Render()` 時被自動設定為 DataForm 的 Id。
