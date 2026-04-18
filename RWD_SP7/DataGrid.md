# DataGrid

> `EEPRWDTools.Core/Controls/DataGrid.cs` — 663 行
> 繼承：`RWDControl` → `Component`
> 介面：`IDataOption`

## 用途

**資料表格元件**（DataGrid）。

DataGrid 是 EEP Core RWD 前端最核心的元件，負責以表格形式顯示、編輯、新增、刪除資料。設計師在模組 JSON 中以 `"type": "datagrid"` 宣告，定義欄位（Columns）、工具列按鈕（ToolItems）、查詢欄位（QueryColumns）、關聯父子元件等設定。Runtime 時渲染為 Bootstrap 表格，搭配分頁、排序、行內編輯、查詢面板等功能。

主要特色：
- 欄位定義（Column）支援格式化、編輯器、關聯、合計
- 內建工具列按鈕（新增/編輯/刪除/查詢/匯入/匯出/列印等）
- 查詢面板（QueryColumn）支援多種查詢模式（Dialog / Panel / Fuzzy）
- AUD 權限偵測（AUD_Detect）整合安全群組/使用者權限
- 父子關聯（ParentObject / TargetObject / WhereItems）
- 欄位可見性由 `[Security]` 屬性控制

## JSON 設定範例

```json
{
  "type": "datagrid",
  "id": "dgMain",
  "remoteName": "cmdMain",
  "editForm": "dfEdit",
  "pagination": true,
  "pageSize": 20,
  "showCheckbox": false,
  "editOnEnter": true,
  "bordered": true,
  "hover": true,
  "toolItemPosition": "Top",
  "columns": [
    { "field": "USERID", "title": "User ID", "cWidth": 120, "sortable": true },
    { "field": "USERNAME", "title": "User Name", "cWidth": 200 },
    { "field": "DEPT", "title": "Department", "format": "N0", "alignment": "right" }
  ],
  "toolItems": [
    { "text": "Add", "iconCls": "glyphicon-plus", "onclick": "insert_row" },
    { "text": "Delete", "iconCls": "glyphicon-remove", "onclick": "delete_row" },
    { "text": "Export", "iconCls": "glyphicon-export", "onclick": "export" }
  ],
  "queryColumns": [
    { "field": "USERNAME", "title": "User Name", "operator": "%%" }
  ]
}
```

## 設計介面屬性

### DataGrid 主要屬性

| 屬性 | 類型 | 預設值 | 設計介面 | 說明 |
|------|------|--------|----------|------|
| **Id** | string | — | 文字 | 元件識別碼（繼承自 Component） |
| **RemoteName** | string | — | 命令選擇器 `[CommandEditor]` | 綁定的伺服器端命令 ID |
| **ReportName** | string | — | 選單選擇器 `[MenuEditor("report")]` | 報表名稱 |
| **ReportType** | ReportType | Pdf | 下拉 `[DataOption]` | 報表類型：`Pdf` / `Excel` / `Preview` |
| **Columns** | List\<Column\> | [] | 集合編輯器 `[CollectionEditor("bootstrap","datagrid","column")]` | 欄位定義集合 |
| **WhereStr** | string | — | 文字 | 固定 WHERE 條件字串 |
| **AlwaysClose** | bool | false | 勾選 | 是否總是關閉編輯狀態 |
| **AutoApply** | bool | true | 勾選 `[CheckboxEditor(true)]` | 自動套用變更 |
| **ValidateXss** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否驗證 XSS |
| **ConfirmDelete** | bool | true | 勾選 `[CheckboxEditor(true)]` | 刪除前是否確認 |
| **DuplicateCheck** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否檢查重複 |
| **EditOnEnter** | bool | true | 勾選 `[CheckboxEditor(true)]` | 按 Enter 是否進入編輯 |
| **FixColumnHeader** | bool | false | 勾選 | 是否固定欄位標題 |
| **ToolItems** | List\<ToolItem\> | [] | 集合編輯器 `[CollectionEditor("bootstrap","datagrid","toolItem","text")]` | 工具列按鈕集合 |
| **ToolItemPosition** | ToolItemPosition | Top | 下拉 | 工具列位置：`Top` / `Bottom` |
| **ViewCommandVisible** | bool | false | 安全控制 `[Security("id")]` | 檢視按鈕可見性 |
| **EditCommandVisible** | bool | false | 安全控制 `[Security("id")]` | 編輯按鈕可見性 |
| **DeleteCommandVisible** | bool | false | 安全控制 `[Security("id")]` | 刪除按鈕可見性 |
| **AUD_Detect** | bool | false | 勾選 | 啟用 AUD 權限偵測（新增/修改/刪除） |
| **ColumnHidable** | bool | false | 勾選 | 欄位是否可隱藏 |
| **EditForm** | string | — | 控制項選擇器 `[ControlEditor({"dataform"})]` | 編輯表單元件 ID |
| **ParentObject** | string | — | 控制項選擇器 `[ControlEditor({"datagrid","dataform"})]` | 父元件 ID |
| **TargetObject** | string | — | 控制項選擇器 `[ControlEditor({"datagrid","dataform"})]` | 目標元件 ID |
| **WhereItems** | List\<WhereItem\> | [] | 集合編輯器 `[CollectionEditor("bootstrap","datagrid","whereItem")]` | 父子關聯條件 |
| **TotalMode** | TotalMode | Page | 下拉 | 合計模式：`Page`（當頁）/ `All`（全部） |
| **ShowColumnTitle** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否顯示欄位標題 |
| **AutoQueryColumn** | bool | false | 勾選 | 自動查詢欄位 |
| **QueryColumns** | List\<QueryColumn\> | [] | 集合編輯器 `[CollectionEditor("bootstrap","datagrid","queryColumn")]`，安全控制 `[Security(false)]` | 查詢欄位集合 |
| **QueryMode** | QueryMode | Dialog | 下拉 | 查詢模式：`Dialog` / `Panel` / `Fuzzy` |
| **QueryTitle** | string | — | 文字 `[Localization("id")]` | 查詢面板標題（支援多語系） |
| **QueryColumnsCount** | int | 2 | 數字框 `[NumberboxEditor(1, true, 2)]` | 查詢面板欄數 |
| **ShowCheckbox** | bool | false | 勾選 | 是否顯示勾選框 |
| **CheckOnSelect** | bool | false | 勾選 | 選取列時是否自動勾選 |
| **RecordLock** | bool | false | 勾選 | 是否啟用記錄鎖定 |
| **Pagination** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否啟用分頁 |
| **PageCount** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否顯示頁數 |
| **PageJump** | bool | false | 勾選 `[CheckboxEditor(false)]` | 是否顯示跳頁 |
| **Rownumbers** | bool | false | 勾選 | 是否顯示列號 |
| **PageSize** | int | 10 | 數字框 `[NumberboxEditor(1, true, 10)]` | 每頁筆數 |
| **PageList** | JArray | [] | 陣列編輯器 `[ArrayEditor("int")]` | 可選頁筆數清單 |
| **Title** | string | — | 文字 `[Localization("id")]` | 表格標題（支援多語系） |
| **Height** | int? | null | 數字框 `[NumberboxEditor(100)]` | 表格高度（px） |
| **Bordered** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否顯示框線 |
| **Hover** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否啟用 hover 效果 |
| **Striped** | bool | false | 勾選 | 是否啟用斑馬紋 |
| **Condensed** | bool | false | 勾選 | 是否啟用緊湊模式 |
| **Xsblock** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否啟用 xs 斷點 block |
| **Simple** | bool | false | 勾選 | 是否啟用簡易模式 |
| **chatDetect** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否偵測聊天 |

## 事件

所有事件均以 `[ScriptEditor]` 定義，在設計介面中以腳本編輯器呈現。

| 事件 | 參數 | 預設回傳 | 說明 |
|------|------|----------|------|
| **RowStyler** | `index, row` | — | 列樣式函式，可依列資料動態設定樣式 |
| **OnBeforeLoad** | `param` | — | 載入前觸發，可修改查詢參數 |
| **OnLoad** | `data` | — | 載入完成後觸發 |
| **OnLoadError** | `msg` | — | 載入錯誤時觸發 |
| **OnSelect** | `index, row` | — | 選取列時觸發 |
| **OnInsert** | `row` | `return true;` | 新增前觸發，回傳 false 可取消 |
| **OnUpdate** | `row` | `return true;` | 修改前觸發，回傳 false 可取消 |
| **OnDelete** | `row` | `return true;` | 刪除前觸發，回傳 false 可取消 |
| **OnDeleted** | `row` | — | 刪除後觸發 |
| **OnApplied** | `data` | — | 套用（Apply）後觸發 |
| **OnQuery** | `whereItems` | `return true;` | 查詢前觸發，回傳 false 可取消 |
| **OnImportExcelSuccess** | （無） | — | Excel 匯入成功後觸發 |
| **OnShowEditor** | `index, field, editor` | `return editor;` | 顯示編輯器前觸發，可替換編輯器 |
| **OnEndEdit** | `index, row` | — | 結束編輯時觸發 |
| **OnTotal** | `totalRow, row` | — | 合計計算時觸發，可自訂合計邏輯 |

## 前端使用範例

### OnBeforeLoad — 動態設定過濾條件

```javascript
function dgMain_onBeforeLoad(data) {
    data.whereItems = JSON.stringify([{
        field: 'USERID',
        operator: '=',
        value: $.fn.clientInfo.user
    }]);
}
```

### OnSelect — 主從連動（選取主表行時刷新從表）

```javascript
function dgMain_onSelect(index, row) {
    $('#dgDetail').datagrid('load', {
        whereItems: JSON.stringify([{
            field: 'ORDER_NO',
            operator: '=',
            value: row.ORDER_NO
        }])
    });
}
```

### OnTotal — 加總完成後顯示

```javascript
function dgMain_onTotal(data) {
    alert('合計金額：' + data.AMOUNT);
}
```

## JavaScript 實戰範例（討論區彙整）

以下範例與陷阱取自 EEP 討論區（`https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=XXX`），都是真實客戶場景。

### 1. `autoApply` 是資料寫入 DB 的關鍵開關

> 來源：[#482525](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482525)

**症狀**：DataGrid 新增後資料顯示在最下方，但沒寫進 DB。

**原因**：`autoApply` 屬性預設 false 時，資料只留在前端變更追蹤清單（`insertedRows` / `updatedRows` / `deletedRows`），要呼叫 `submit` 或在主表 DataForm 一起送出才真的寫 DB。

```javascript
// JSON 設計時設
{
    "type": "datagrid",
    "id": "dgMaster",
    "autoApply": true   // ← 新增即寫 DB
}

// 或程式化
$('#dgMaster').datagrid('options').autoApply = true;

// 或手動 submit 變更
$('#dgMaster').datagrid('submit');
```

### 2. 沒有 onApply 事件 → 用 Validate 元件

> 來源：[#482523](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482523)

DataGrid **沒有** `onApply` 事件。要在送出前檢核欄位（例如 Total 不能超過某值）：

- **不要**試著攔 apply 時機
- **應**在同容器放一個 `Validate` 元件，`BindingObject` 指向此 DataGrid，設定欄位檢核規則
- 系統會自動阻擋不合規的 apply

### 3. 虛擬欄位用 formatter 呼叫後端運算

> 來源：[#482511](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482511)

**錯誤做法**：在 `onLoad` 逐筆 `callServerMethod` + `updateRow`（耗效能、會觸發變更追蹤）。

**正確做法**：用欄位 `formatter` 直接 return 運算值（formatter 是顯示層，不會汙染變更追蹤）：

```javascript
// column 定義
{
    "field": "UNIT_PRICE_2",
    "title": "單價2（虛擬）",
    "formatter": "formatUnitPrice2"
}

// JS
function formatUnitPrice2(value, row, index) {
    // 同步呼叫後端運算
    return $.callSyncMethod('ModuleName', 'CalcPrice2', [row.ITEM_ID, row.QTY]);
}
```

若要取得所有分頁的運算結果：

```javascript
var rows = $('#dgMaster').datagrid('getRows');
for (var i = 0; i < rows.length; i++) {
    console.log(rows[i].UNIT_PRICE_2);
}
```

### 4. 依欄位值動態控制 Edit / Delete 按鈕

> 來源：[#482346](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482346) / [#482420](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482420)

**建議做法**：在 `onUpdate` / `onDelete` 時機點判斷 + `return false`，而非一開始就把按鈕 disable：

```javascript
function dgMaster_onUpdate(row) {
    // 已關帳月份不可編輯
    var ym = row.進貨日期.substr(0, 4) + row.進貨日期.substr(5, 2);
    var closeYm = $.getVariableValue('closeYm');
    if (ym <= closeYm) {
        $.alert('已關帳月份不可編輯', 'warning');
        return false;  // ← 阻止進入編輯模式
    }
    return true;
}

function dgMaster_onDelete(row) {
    if (row.UseStatus === '後端已使用') {
        $.alert('已被引用不可刪除', 'warning');
        return false;
    }
    return true;
}
```

優點：按鈕本身不用動、程式碼集中、錯誤訊息清楚。

### 5. ParentObject 後 QueryColumns 失效（限制 + 解法）

> 來源：[#482368](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482368)

**限制**：主明細架構下，明細 DataGrid 的 `QueryColumns` **不會生效**（系統不支援）。

**解法**：把 server 端 `infoDataSource` 關聯拿掉，改由前端控制：

```javascript
// 明細 onBeforeLoad 動態抓主檔 key
function dgDetail_onBeforeLoad(param) {
    var masterRow = $('#dgMaster').datagrid('getSelected');
    if (masterRow) {
        param.whereStr = "ORDER_NO = '" + masterRow.ORDER_NO + "'";
    }
}
```

注意：這樣做**新增/編輯/刪除的關聯處理要自行補**（原本 server 端會自動處理主明細一致性）。

### 6. 完全隱藏 DataGrid（含 toolitem 和分頁）

> 來源：[#482103](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482103)

```javascript
// ❌ 只隱藏表體，toolitem 和分頁還在
$('#dgMaster').hide();

// ✅ 隱藏整個 DataGrid 容器（含上方工具列與下方分頁）
$('#dgMaster').closest('div').parent().hide();

// 顯示回來
$('#dgMaster').closest('div').parent().show();
```

原因：DataGrid 的工具列、主體、分頁三段在 DOM 上是同層兄弟，都在同一個父層 `<div>` 下。

### 7. setWhere 後強制選第一筆

> 來源：[#481901](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481901)

```javascript
$('#dgMaster').datagrid('setWhere', "STATUS = 'A'");

// setWhere 是非同步，必須 setTimeout 等 reload 完成
setTimeout(function () {
    var $trs = $('#dgMaster').find('tr');
    $trs.removeClass('selected info');
    $trs.eq(1).addClass('selected info');  // eq(1) 才是第一筆（eq(0) 是 header）
    // 觸發 onSelect（可選）
    var row = $('#dgMaster').datagrid('getRows')[0];
    if (row) {
        var opts = $('#dgMaster').datagrid('options');
        if (opts.onSelect) opts.onSelect.call(null, 0, row);
    }
}, 300);
```

### 8. 分辨使用者是「新增」還是「編輯」

> 來源：[#482208](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482208)

DataGrid 沒有直接取得「當前模式」的 API。用 flag 變數配合事件：

```javascript
var editMode = '';

function dgMaster_onInsert(row)     { editMode = 'insert'; }
function dgMaster_onUpdate(row)     { editMode = 'update'; }

function dgMaster_onShowEditor(index, field, editor) {
    if (editMode === 'insert' && (field === 'A' || field === 'B' || field === 'C')) {
        // 新增時這三欄可編
        $('#dgMaster').datagrid('setEditorReadonly', { field: field, value: false });
    } else if (editMode === 'update') {
        $('#dgMaster').datagrid('setEditorReadonly', { field: field, value: true });
    }
}
```

### 9. `onShowEditor` 給值的時機陷阱

> 來源：[#482457](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482457)

`onShowEditor` 觸發時，**編輯器可能還沒完全渲染**。立即 `setValue` 會失效。

```javascript
function dg_onShowEditor(index, field, editor) {
    if (field === 'FIRST') {
        // ❌ 可能無效
        $('#dg').datagrid('setEditorValue', { field: 'FIRST', value: 'X' });

        // ✅ 延後一個 tick
        setTimeout(function () {
            $('#dg').datagrid('setEditorValue', { field: 'FIRST', value: 'X' });
        }, 0);
    }
}
```

### 10. 兩個 DataGrid 關聯顯示 — 用 server infoDataSource

> 來源：[#481965](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481965)

**情境**：`dgDetail.Detail_ATN = dgMaster.AutoNo`，希望 dgDetail 自動跟著主表顯示。

**做法**：不用在前端寫 `onLoad` 過濾。在 **server 端加 `infoDataSource` 關聯**即可，系統自動處理主從資料同步。

### 11. 三階 DataGrid 聯動設定

> 來源：[#481904](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481904)

三階結構 A → B → C：
- A/B 關聯欄位：`A1`
- B/C 關聯欄位：`A1, B1`（**兩個 key 都要設**）

三階結構系統是透過**前端快取**處理的。只要 server 端 `infoDataSource` 的雙 key 都設對，切換 A 選列不會亂跳。

**限制**：DataGrid 的**複製按鈕目前只支援到二階**（[#482171](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482171)），三階要靠 JS 自己實作複製。

### 12. 取得「所有分頁」的資料（而非當前頁）

> 來源：[#481910](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481910)

`$('#dgMaster').datagrid('getRows')` **只回傳當前頁的資料**。要取全部：

```javascript
// ❌ 只有當前頁
var rows = $('#dgMaster').datagrid('getRows');

// ✅ 呼叫後端取全部（避免前端載入過多）
var allRows = $.callSyncMethod('ModuleName', 'GetAllRows', [queryParams]);
```

前端**不建議**強制載入全部資料做運算 — 效能差、安全性低。聚合運算應在後端處理。

### 13. 越南文 / 泰文 / 特殊字元無法顯示

> 來源：[#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032)

**症狀**：輸入時可正常顯示，`確定`後欄位變空白。

**原因**：XSS 驗證過濾掉特殊字元。

**解法**：DataGrid + UpdateComponent **雙邊都要關**：

```json
{
  "type": "datagrid",
  "id": "dgMaster",
  "validateXss": false
}
```

```json
{
  "type": "updatecomponent",
  "id": "ucMaster",
  "validateXss": false
}
```

### 14. 動態設定 Height

> 來源：[#481941](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481941)

**症狀**：`$('#dgMaster').datagrid('options').Height = webheight` 沒有反應。

**原因**：設 options 不會觸發重繪。

**解法**：直接改容器 CSS + 讓 datagrid 重新計算：

```javascript
$(window).resize(function () {
    var webheight = window.innerHeight - 300;
    $('#dgMaster').closest('.bootstrap-datagrid').css('max-height', webheight + 'px');
});
```

### 常見陷阱（討論區彙整）

| 陷阱 | 說明 | 解法 | 來源 |
|------|------|------|------|
| **新增不寫 DB** | `autoApply=false` 時只追蹤變更 | 設 `autoApply=true` 或呼叫 `submit()` | [#482525](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482525) |
| **沒有 `onApply` 事件** | 系統設計上無此時機點 | 用 `Validate` 元件阻擋 | [#482523](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482523) |
| **虛擬欄位用 `updateRow` 效能差** | 會觸發變更追蹤 | 用 `formatter` return | [#482511](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482511) |
| **明細 `QueryColumns` 失效** | 主明細架構不支援 | 拆開 `infoDataSource` + JS 控制 whereStr | [#482368](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482368) |
| **`hide()` 只藏表體** | 工具列/分頁還在 | `closest('div').parent().hide()` | [#482103](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482103) |
| **`setWhere` 後立刻 `getRows` 拿舊資料** | setWhere 是非同步 | `setTimeout` 300ms | [#481901](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481901) |
| **`onShowEditor` setValue 失效** | 編輯器渲染未完成 | `setTimeout(..., 0)` | [#482457](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482457) |
| **越南文/特殊字元變空白** | XSS 驗證過濾 | DataGrid + UpdateComponent 都關 `validateXss` | [#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032) |
| **`getRows` 只有當前頁** | 前端分頁特性 | 用後端 `callSyncMethod` 取全部 | [#481910](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481910) |
| **複製按鈕第三階不支援** | 系統限制 | JS 手動實作 | [#482171](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482171) |
| **動態 Height 無效** | 改 options 不會重繪 | 改容器 CSS | [#481941](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481941) |
| **判斷新增或編輯** | 無直接 API | flag 變數 + onInsert/onUpdate | [#482208](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482208) |

## 內部類別

### Column

> 欄位定義，繼承 `RWDCollectionItem`，實作 `IDataOption`

| 屬性 | 類型 | 預設值 | 設計介面 | 說明 |
|------|------|--------|----------|------|
| **Title** | string | "" | 文字 `[Localization("field")]` | 欄位標題（支援多語系） |
| **Field** | string | — | 欄位選擇器 `[ColumnEditor]` | 對應資料欄位名稱 |
| **CWidth** | int? | null | 數字框 `[NumberboxEditor(40)]`，顯示為 "width" `[Title("width")]` | 欄位寬度（px） |
| **AlignTop** | string | — | 欄位選擇器 `[ColumnEditor("", false, "alignTopTitle")]` | 上層對齊欄位 |
| **Alignment** | string | — | 下拉 `[ItemsEditor({"left","center","right"})]` | 對齊方式 |
| **Nowrap** | bool | false | 勾選 | 是否禁止換行 |
| **Hidden** | bool | false | 安全控制 `[Security("field")]` | 是否隱藏 |
| **Sortable** | bool | false | 勾選 | 是否可排序 |
| **Showxs** | bool | true | 勾選 `[CheckboxEditor(true)]` | xs 斷點是否顯示 |
| **Showsm** | bool | true | 勾選 `[CheckboxEditor(true)]` | sm 斷點是否顯示 |
| **Format** | string | — | 下拉 `[ItemsEditor]`（可自訂） | 格式化：`N0` / `yyyy/MM/dd` / `logic` / `checkbox` / `image` / `file` / `barcode` / `qrcode` / `map` / `signature` / `drilldown` / `flowflag` / `creator` / `updater` |
| **Formatter** | string | — | 腳本編輯器 `[ScriptEditor({"value","row","index"}, "return value;")]` | 自訂格式化函式 |
| **Total** | Total | None | 下拉 | 合計方式：`None` / `Sum` / `Max` / `Avg` |
| **Editor** | RWDEditor | Textbox | 編輯器選擇器 `[EditorOptionEditor("bootstrap","","textbox")]` | 行內編輯器 |
| **Relation** | Relation | Relation() | 關聯編輯器 `[EditorOptionEditor("bootstrap","relation")]` | 欄位關聯設定 |

### QueryColumn

> 查詢欄位定義，繼承 `RWDCollectionItem`，實作 `IDataOption`、`IFormColumn`

| 屬性 | 類型 | 預設值 | 設計介面 | 說明 |
|------|------|--------|----------|------|
| **Title** | string | — | 文字 `[Localization("title")]` | 查詢欄位標題（支援多語系） |
| **Field** | string | — | 欄位選擇器 `[ColumnEditor("",false,"","query")]` | 對應資料欄位名稱 |
| **NewRow** | bool | true | 勾選 `[CheckboxEditor(true)]` | 是否另起新列 |
| **Span** | int | 1 | 數字框 `[NumberboxEditor(1, true, 1)]` | 佔用欄數 |
| **Operator** | string | "=" | 運算子選擇器 `[OperatorEditor]` | 查詢運算子 |
| **Editor** | RWDEditor | Textbox | 編輯器選擇器 `[EditorOptionEditor("bootstrap","","textbox")]` | 查詢輸入編輯器 |
| **DefaultValue** | string | — | 預設值編輯器 `[ValueOptionEditor("bootstrap","default")]` | 查詢預設值 |

### ToolItem

> 工具列按鈕定義，實作 `IToolItem`

| 屬性 | 類型 | 預設值 | 設計介面 | 說明 |
|------|------|--------|----------|------|
| **Text** | string | — | 文字 `[Localization]` | 按鈕文字（支援多語系） |
| **HelpText** | string | — | 文字 `[Localization]` | 提示文字（popover，支援多語系） |
| **IconCls** | string | — | 圖示選擇器 `[IconRWDEditor("bootstrap")]` | 圖示 CSS 類別 |
| **IconAlign** | IconAlign | Left | 下拉 | 圖示位置：`Left` / `Right` |
| **BtnCls** | string | — | 按鈕樣式選擇器 `[BtnClsEditor]` | 按鈕 CSS 類別 |
| **Hidden** | bool | false | 安全控制 `[Security("text")]` | 是否隱藏 |
| **Onclick** | string | — | 文字 `[DataOption]` | 點擊動作 |

### 預設 ToolItem 動作

`GetToolItems()` 提供以下預設工具列按鈕：

| Text | IconCls | Onclick | 說明 |
|------|---------|---------|------|
| Add | glyphicon-plus | `insert_row` | 新增列 |
| Edit | glyphicon-edit | `edit_row` | 編輯列 |
| Delete | glyphicon-remove | `delete_row` | 刪除列 |
| Query | glyphicon-search | `openQuery` | 開啟查詢面板 |
| Copy | glyphicon-copy | `copy_row` | 複製列 |
| Ok | glyphicon-ok | `endEdit` | 結束編輯 |
| Cancel | glyphicon-remove | `cancelEdit` | 取消編輯 |
| Import | glyphicon-import | `importExcel` | 匯入 Excel |
| Export | glyphicon-export | `export` | 匯出 |
| Print | glyphicon-print | `exportReport` | 列印報表 |
| UserDefined | glyphicon-print | `userDefined` | 自訂按鈕 |
| UserPrint | glyphicon-print | `userPrint` | 使用者列印 |

### WhereItem

> 父子關聯條件定義，實作 `IDataOption`

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **TargetField** | string | 欄位選擇器 `[ColumnEditor("target")]` | 目標欄位（子元件） |
| **Operator** | string | 運算子選擇器 `[OperatorEditor]`，預設 `"="` | 比對運算子 |
| **SourceField** | string | 欄位選擇器 `[ColumnEditor]` | 來源欄位（父元件） |

## AUD 權限偵測

當 `AUD_Detect = true` 時，DataGrid 在渲染前會檢查當前使用者的安全群組（secGroups）和使用者權限（secUsers），決定是否允許新增（allowAdd）、修改（allowUpdate）、刪除（allowDelete）：

- 若不允許修改：隱藏 EditCommandVisible，停用 EditOnEnter
- 若不允許刪除：隱藏 DeleteCommandVisible
- 工具列中 `insert_row`、`edit_row`、`delete_row` 按鈕依權限自動隱藏

## 渲染流程

```
Render()
  → RenderQuery()           // 渲染查詢面板（QueryColumns > 0 時）
  → div.table-responsive
    → 標題（Title）
    → ToolItems（Top 位置時）
    → RenderTable()          // 渲染 table，套用 Bootstrap CSS 類別
      → thead > tr > th      // 各 Column 渲染為 th
      → tbody                // 空的，由前端填充
    → ToolItems（Bottom 位置時）
  → div.clearfix > ul.pagination  // 分頁列
```

表格 CSS 類別依屬性動態組合：`table-bordered`、`table-hover`、`table-striped`、`table-condensed`、`table-xsblock`、`table-simple`。

## 前端行為（JavaScript）

> 原始碼：`EEPWebClient.Core/wwwroot/js/infolight/bootstrap.infolight.js` 第 2308–6258 行
> jQuery Plugin 名稱：`$.fn.datagrid`，以 `$.createObj('datagrid', { ... })` 建立
> CSS 選擇器：`.bootstrap-datagrid, .bootstrap-tablegrid`

### 渲染後 HTML 結構

伺服器端 `Render()` 產出的 HTML 骨架經前端 `init` 方法加工後，最終 DOM 結構如下：

```
div.table-responsive
  div.datagrid-toolitem              ← 工具列按鈕（Top 位置時在 table 之前）
  div.table-wrapper                  ← 當 height 有設定時包裹，overflow:auto
    table.bootstrap-datagrid         ← 主表格，id 對應元件 ID
      thead > tr
        th.command                   ← 行指令欄（view/edit/delete 按鈕）
        th.rowcheck                  ← showCheckbox 時加入全選 checkbox
        th.rownumber                 ← rownumbers 時加入列號欄
        th[data-field]               ← 各資料欄位，含 sort-btn、popover 隱藏按鈕等
        tr.autoquerytr               ← autoQueryColumn 時動態插入的快速查詢列
      tbody
        tr.dataRow                   ← 每筆資料一組 tr（trLength > 1 時多列一組）
          td.datagrid-rownumber      ← 列號
          td.datagrid-checkbox       ← 勾選框
          td.datagrid-command        ← 行指令按鈕（pencil/remove/eye-open/showCommand）
          td[data-field]             ← 資料儲存格
            b.table-cell-label       ← RWD xs 斷點時顯示的欄位標題
            div.table-cell-text      ← 實際值或編輯器容器
      tfoot                          ← 合計列（由 reloadFooter 動態建立）
  div.datagrid-toolitem              ← 工具列按鈕（Bottom 位置時在 table 之後）
div.clearfix
  ul.pagination                      ← 分頁按鈕列
  div.pageTo                         ← pageJump 時的跳頁輸入框
  div.pageList                       ← pageList 時的每頁筆數下拉選單
```

每筆資料以 `tr.dataRow` 為主列，透過 `$(tr).data('row')` 儲存該列的 JSON 物件。當 Column 設定了 `alignTop`（多列欄位）時，`trLength > 1`，一筆資料由多個 `<tr>` 組成。

### 主要公開 API 方法

以下方法皆透過 `$('#dgId').datagrid('methodName', param)` 呼叫：

| 方法 | 參數 | 說明 |
|------|------|------|
| `init` | `options?` | 初始化元件：解析 `data-options`、建立欄位標題、繫結事件、呼叫 `load` |
| `load` | `{ page }?` | 向伺服器請求資料（`$.loadData`），觸發 `onBeforeLoad` → 取得資料 → `loadData` |
| `loadData` | `{ rows, total, keys }` | 將資料陣列渲染至 tbody，處理 relation 欄位的非同步查詢後批次插入 |
| `reload` | — | 等同 `load`（分頁元件的 refresh 按鈕呼叫） |
| `setWhere` | `whereItems[] \| whereStr` | 設定查詢條件並重新載入（支援陣列或字串） |
| `getRows` | — | 回傳目前 tbody 中所有資料列的 JSON 陣列 |
| `getSelected` | — | 回傳目前選取列的 JSON 物件，無選取時回傳 `null` |
| `getSelectedIndex` | — | 回傳選取列的索引（`tr.selected.dataRow` 的 index / trLength） |
| `getRow` | — | 回傳目前正在編輯中的列資料（含編輯器當前值） |
| `getRowByIndex` | `index` | 依索引取得列的 JSON 物件 |
| `getChecked` | — | 回傳所有勾選列的 JSON 陣列 |
| `getTotal` | — | 回傳 tfoot 合計列的 JSON 物件 |
| `select` | `index` | 選取指定索引的列（加上 `.selected.info` CSS） |
| `check` / `uncheck` | `index` | 勾選/取消勾選指定列 |
| `checkAll` / `uncheckAll` | — | 全選/取消全選 |
| `beginEdit` | `index` | 對指定列啟動行內編輯：為每個 td 建立對應編輯器 |
| `endEdit` | `callback?` | 結束編輯：驗證 → `updateRow` → 若 `autoApply` 則自動 `submit` |
| `cancelEdit` | `callback?` | 取消編輯：若為新增列則移除該列，否則還原顯示 |
| `insert_row` | — | 新增列：取得預設值 → `appendRow` → `beginEdit` |
| `edit_row` | `index?` | 編輯列：檢查 readonly → 觸發 `onUpdate` → `beginEdit` 或開啟 `editForm` |
| `delete_row` | `index?` | 刪除列：確認對話框 → 加入 `deletedRows` → 若 `autoApply` 則 `submit` |
| `copy_row` | `prompt?` | 複製列：以選取列為範本（排除 keys 欄位）建立新資料 |
| `appendRow` | `row` | 在 tbody 末尾插入一列（不啟動編輯） |
| `removeRow` | `index` | 從 DOM 移除指定列並重新編號 |
| `updateRow` | `{ index, row }` | 更新指定列的資料與顯示（同時追蹤至 `updatedRows`） |
| `submit` | `success?, error?` | 將 `insertedRows`/`updatedRows`/`deletedRows` 透過 `$.updateData` 送出 |
| `acceptChanges` | `isLoad?` | 清空變更追蹤（`insertedRows`/`updatedRows`/`deletedRows`） |
| `getChangedDatas` | — | 回傳含 inserted/updated/deleted 的變更資料陣列（含子表） |
| `validate` | `row?` | 檢查主鍵重複（`duplicateCheck`） |
| `validateRow` | `index` | 驗證指定列所有欄位（必填、XSS、自訂 validType），失敗顯示錯誤 |
| `readonly` | `true/false` | 切換唯讀模式：隱藏/顯示編輯相關按鈕，加上 `table-readonly` CSS |
| `setEditorValue` | `{ field, value }` | 設定編輯中指定欄位的編輯器值 |
| `getEditorValue` | `field` | 取得編輯中指定欄位的編輯器值 |
| `setEditorReadonly` | `{ field, value }` | 設定編輯中指定欄位的唯讀狀態 |
| `getColumnOption` | `field` | 取得欄位的完整設定物件（含 width, trIndex 等） |
| `setColumnTitle` | `{ field, title }` | 動態更改欄位標題 |
| `showColumn` / `hideColumn` | `field` | 動態顯示/隱藏欄位（設定存入 `localStorage`） |
| `getToolItem` | `name` | 依 onclick 名稱取得工具列按鈕的 jQuery 物件 |
| `openQuery` | — | 開啟查詢面板（Dialog 模式時以 modal 顯示） |
| `export` | `options?` | 匯出 Excel：收集欄位 → `$.exportData` |
| `exportReport` | `options?` | 列印報表：`$.exportFile('report', ...)` |
| `exportWord` | `options?` | 匯出當前選取列為 Word/PDF（單筆）｜見下方[Word 匯出方法](#word-匯出方法) |
| `exportWordPdf` | `name, password?, watermark?` | 當前選取列匯出 PDF（便捷） |
| `exportWordExcel` | `name` | 以 Word 範本邏輯匯出為 Excel (.xlsx) |
| `exportWordExcelPdf` | `name` | Excel 範本轉 PDF |
| `exportWordAll` | `options?` | 批次：依查詢條件每筆各產生一檔，打包成 ZIP |
| `exportWordLoop` | `options?` | 批次：多筆合併為單一 Word 檔（迴圈範本） |
| `exportWordLoopPdf` | `name` | `exportWordLoop` 的 PDF 版 |
| `importExcel` | `options?` | 匯入 Excel：彈出上傳對話框 → `$.ajaxSubmit` 至 `../excelfile` |
| `getDetailGrid` | `excludeForm?` | 取得以此 DataGrid 為 `parentObject` 的所有子 DataGrid |
| `getParentGrid` | — | 取得以此 DataGrid 為 `targetObject` 的父 DataGrid |
| `getParentValues` | — | 從父元件取得關聯鍵值（用於新增子列時帶入父鍵） |

### Word 匯出方法

DataGrid 提供 3 種 Word 匯出模式（+ 3 個 PDF 便捷變體、1 個 Excel 變體），全部走相同的後端 `$.exportFile` 機制：

| 模式 | 資料範圍 | 輸出 | 對應後端 |
|------|----------|------|----------|
| `exportWord` | 當前選取的單一列 + 明細 | 單一 Word 檔 | `ParserHelper.ExportWord` |
| `exportWordLoop` | 符合查詢條件的多筆（合併到同一範本的迴圈） | 單一 Word 檔 | `ParserHelper.ExportWordLoop` |
| `exportWordAll` | 符合查詢條件的多筆（各自產檔） | ZIP 壓縮檔 | `ParserHelper.ExportWordAll` |

#### exportWord(options)

匯出「當前選取列」為 Word (.docx)。呼叫前 **必須先用 `selectRow` 或使用者點選**讓 `getSelectedIndex() >= 0`，否則跳警告。

**options 物件**：

| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `name` | string | 取 `#_PATHNAME` 或 URL 最後一段 | 範本檔名（不含副檔名），後端會到 `design/word/{Solution}/{name}.docx` 找範本 |
| `fileType` | string | `""` | 設 `"pdf"` 轉 PDF；其他值或空 → `.docx` |
| `wordName` / `fileName` | string | — | 覆寫範本檔名（優先於 `name`） |
| `downloadName` | string | `""` | 下載時顯示的檔名（不影響範本查找） |
| `directOpen` | bool | `false` | `fileType='pdf'` 時：true = 瀏覽器內 inline 開啟、false = 下載 |
| `password` | string | — | PDF 密碼保護 |
| `watermark` | string | — | PDF 浮水印文字 |
| `titleName` | string | — | 傳到範本 `${TitleName}` 佔位符（客製版才用） |

**若 options 為字串**，會被包成 `{ name: options }`。

**範例**：

```javascript
// 最簡用法：以當前 URL 路徑為範本名
$('#dgMain').datagrid('exportWord');

// 指定範本名
$('#dgMain').datagrid('exportWord', '訂單報表');

// 完整選項：PDF + 浮水印 + 密碼 + 自訂下載檔名
$('#dgMain').datagrid('exportWord', {
    name: '訂單報表',
    fileType: 'pdf',
    password: '1234',
    watermark: '機密',
    downloadName: 'Order_' + new Date().getTime(),
    directOpen: false
});
```

#### exportWordPdf(name, password, watermark)

`exportWord` 的 PDF 便捷版，等於：

```javascript
$('#dgMain').datagrid('exportWord', { fileType: 'pdf', name, password, watermark });
```

#### exportWordExcel(name)

以 Word 範本的邏輯（`${Field}` 語法）匯出為 **Excel** (.xlsx)。後端 `ParserHelper` 判斷 `filePath.Split(".")[1] == "xlsx"` 時會改呼叫 `Parser.PreviewExcel`。用於 Excel 範本有 `${...}` 佔位符的情境。

```javascript
$('#dgMain').datagrid('exportWordExcel', '訂單明細');  // → 訂單明細.xlsx
```

#### exportWordExcelPdf(name)

`exportWordExcel` 轉 PDF：

```javascript
$('#dgMain').datagrid('exportWordExcelPdf', '訂單明細');
```

#### exportWordAll(options)

**批次匯出：每筆各產一個檔，打包成 ZIP**。以 DataGrid 的 `whereStr` / `whereItems` 查詢所有符合筆資料，對每筆執行一次 Word 範本合成，最後打包。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `name` | string | 範本檔名 |
| `fileType` | string | `"pdf"` 時每個檔都轉 PDF 再打包 |

無 `whereItems` 覆寫選項 — 直接用 DataGrid 當前的查詢條件。

```javascript
// 用當前 DataGrid 的查詢條件，全部匯出為 ZIP
$('#dgMain').datagrid('exportWordAll', '訂單報表');

// PDF 打包
$('#dgMain').datagrid('exportWordAll', {
    name: '訂單報表',
    fileType: 'pdf'
});
```

#### exportWordLoop(options)

**批次合併匯出：多筆資料填入同一範本的迴圈區段，產生單一檔案**。範本中要用 `{#loop}...{/loop}` 或類似迴圈標記。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `name` | string | 範本檔名 |
| `fileType` | string | `"pdf"` 轉 PDF |
| `whereItems` | array | 覆寫查詢條件（若為空陣列 → 用 DataGrid 的 whereItems） |
| `downloadName` | string | 下載顯示檔名 |
| `directOpen` | bool | PDF 直接瀏覽器開啟 |

```javascript
// 用當前查詢條件，合併所有筆到單一 Word
$('#dgMain').datagrid('exportWordLoop', '月報');

// 自訂查詢條件
$('#dgMain').datagrid('exportWordLoop', {
    name: '月報',
    fileType: 'pdf',
    whereItems: [
        { field: 'MONTH', operator: '=', value: '2026-04' },
        { field: 'STATUS', operator: '=', value: 'DONE' }
    ],
    directOpen: true
});
```

#### exportWordLoopPdf(name)

等於 `exportWordLoop({ fileType: 'pdf', name })`。

### Word 匯出請求流程

```
前端：$('#dg').datagrid('exportWord', options)
  ↓ 組 param = { remoteName, masterRow, fileType, wordName, downloadName, ... }
  ↓ $.exportFile('word', name, param, onSuccess, onError, onComplete)

AJAX POST ../main/file
  body: { mode: 'exportFile', type: 'word', id: name, param: JSON.stringify(param) }
  ↓
後端：FileProvider.ExportFile(type, param)
  switch (type)
    case "word"     → ExportWord(id, options)          → ParserHelper.ExportWord
    case "wordLoop" → ExportWordLoop(id, options, qp)  → ParserHelper.ExportWordLoop
    case "wordAll"  → ExportWordAll(id, options, qp)   → ParserHelper.ExportWordAll
  ↓ 回傳產生的檔名
前端 onSuccess(file)
  directOpen && pdf → window.open('../file?q=xxx&t=inline')
  其他             → window.location.href = '../file?q=xxx&n=downloadName'
```

### 範本檔案位置

後端 `ParserHelper` 在以下路徑依序尋找範本：

```
design/word/{Solution}/{name}.docx    ← 優先（站台專屬）
design/word/{name}.docx                ← Fallback（共用）
```

若是 Excel 範本，副檔名改為 `.xlsx`，會觸發 Excel 分支。

### 範本語法（簡述）

| 語法 | 說明 |
|------|------|
| `${FIELD}` | 主表欄位值 |
| `${@FIELD}` | 主表欄位顯示值（經 coldef 轉換） |
| `${@User}` / `${@UserName}` | 登入者 ID / 名稱 |
| `${@Today}` / `${@Now}` | 今日 / 現在時間 |
| `${@Password}` / `${@Watermark}` | options.password / watermark |
| `${@TitleName}` | options.titleName（客製版） |
| 明細迴圈 | 範本定義的表格迴圈區塊（Parser 套件解析） |
| 流程簽核 | `${@W}`、`${@WN}`、`${@WC}` 等（僅 ExportWord，Status=3 抽回會被排除） |

---

### 資料載入流程

```
init()
  → acceptChanges(true)             // 初始化變更追蹤
  → loadColumnSettings()            // 從 localStorage 讀取隱藏欄位設定
  → bindEvent()                     // 繫結 click 事件
  → load()
       ↓
  parentObject 且無 parentTable？ → 中止（等待父元件 select 後再載入）
       ↓
  組裝參數 param { rows, page, whereStr, whereItems, sort, order, ... }
       ↓
  觸發 onBeforeLoad(param) → 回傳 false 可中止載入
       ↓
  loadCache() 嘗試從記憶體快取載入 → 成功則跳過 AJAX
       ↓
  $.loadData(remoteName, param, success, error)   ← AJAX POST
       ↓ success
  acceptChanges(true)
  loadData(data)
    → 清空 tbody
    → 非同步載入 relation 欄位的對照值（$.when + $.getDisplayRowP）
    → 逐列呼叫 insertRow() 建立 DOM
    → refreshPagination(total)       // 更新分頁按鈕
    → createPageTotal(data)          // 計算合計
    → 觸發 loadAction（若有，如 insert_row / edit_row）
    → 觸發 onLoad(data)
    → loadDetail()                   // 載入子表
```

**關鍵細節**：
- `rows` 參數為 `-1` 時表示不分頁（`pagination: false`）
- 當資料量大時（`data.pageSize` 有值），使用 `setTimeout` 分批插入以避免 UI 凍結
- Relation 欄位透過 `$.when` 批次查詢對照值後，以 `relationValues` 快取供 `formatDisplay` 使用

### 編輯流程

#### 行內編輯（無 editForm）

```
點擊列 (tr click)
  → select(index)
  → onSelect(index, row)
  → loadDetail(row)                  // 通知子表
  → editOnEnter 為 true？
      → edit_row(index)
           → endEdit()               // 先結束上一列的編輯
           → beginEdit(index)
                → 為每個 td 建立編輯器（依 column.editor.type）
                → 觸發 onShowEditor(index, field, editor) 可替換編輯器
                → 設定焦點至 focusedField 或第一個可編輯欄位
```

#### 結束編輯

```
endEdit()
  → validateRow(index)              // 逐欄驗證：XSS → 必填 → validType
  → 驗證失敗？ → 顯示錯誤訊息，中止
  → updateRow({ index, row })       // 更新 DOM 與資料
  → 觸發 onEndEdit(index, row)
  → autoApply 為 true？
      → submit()                    // 立即送出 AJAX 儲存
           → $.updateData(remoteName, changedDatas)
           → 成功：acceptChanges() → 觸發 onApplied(data)
```

#### 透過 EditForm 編輯

```
edit_row(index)
  → 有 editForm？
      → $.addLock()                  // recordLock 時先鎖定記錄
      → form.open({ row, status:'updated', keys })
      ← 由 DataForm 元件處理儲存
```

#### 新增列

```
insert_row()
  → readonly？ → 中止
  → getParentValues()                // 取得父元件關聯鍵值
  → getDefaultValues(parentRow)      // 計算預設值（default/carryOn/chatUX）
  → 觸發 onInsert(defaultRow) → 回傳 false 可取消
  → 有 editForm？
      → form.open({ row, status:'inserted' })
  → 無 editForm？
      → endEdit() → appendRow(defaultRow) → beginEdit(lastIndex)
      → 新列加入 opts.insertedRows
```

#### 刪除列

```
delete_row(index)
  → readonly？ → 中止
  → 觸發 onDelete(row) → 回傳 false 可取消
  → confirmDelete 為 true？ → $.confirm 確認對話框
  → 該列為新增列（在 insertedRows 中）？
      → 直接 removeRow，不送 AJAX
  → 該列為既有資料？
      → $.addLock()（recordLock 時）
      → 加入 opts.deletedRows
      → autoApply？ → submit() → removeRow()
```

### 工具列按鈕對應

工具列按鈕的 `onclick` 值對應到 `$.fn.datagrid.methods` 中的方法名稱。若找不到對應方法，則呼叫 `$.callFunction(onclick)` 作為全域函式執行。按鈕點擊後會短暫 disable 200ms 防止重複觸發。

### 主從連動機制（Master-Detail）

1. **父子宣告**：子 DataGrid 設定 `parentObject` 指向父元件 ID，`whereItems` 定義關聯欄位對應
2. **init 階段**：若 `parentObject` 有設定但尚未有 `parentTable`，init 會中止載入（`return true`），等待父元件觸發
3. **父元件 select**：父元件列被點擊時，`bindEvent` 中的 tr click handler 會：
   - 組裝 `whereItems`（將 `targetWhereItems` 的 `sourceField` 替換為實際值）
   - 呼叫子元件的 `setWhere(whereItems)` 重新載入
4. **loadDetail**：父元件載入資料後呼叫 `loadDetail()`，傳遞 `parentTable`、`parentRow`、`parentKeys` 給子表
5. **快取機制**：子表透過 `saveCache` / `loadCache` 以父鍵值為 key 快取資料，切換父列時先存再取

### 分頁行為

- 分頁按鈕由 `refreshPagination(total)` 動態建立於 `ul.pagination` 中
- 顯示邏輯：目前頁 ±2 頁的頁碼按鈕 + 首頁/末頁/上頁/下頁 + 重新整理
- `pageCount: false` 時隱藏總頁數，僅顯示上頁/下頁（適用於大量資料）
- `pageJump: true` 時顯示跳頁輸入框，Enter 鍵觸發載入
- `pageList` 有值時顯示每頁筆數下拉選單，切換時自動重新載入

### 排序行為

- 可排序欄位的 `<th>` 上有 `.sort-btn`（`fa fa-sort`）
- 點擊切換 asc/desc，將 `opts.sort` 與 `opts.order` 帶入 `load()` 參數送至伺服器端排序

### 查詢面板行為

| 查詢模式 | 前端行為 |
|----------|----------|
| `dialog` | `openQuery()` 以 Bootstrap modal 開啟查詢表單 |
| `panel` | `initQuery()` 初始化內嵌面板，載入預設值 |
| `autoQueryColumn` | 在 thead 插入 `tr.autoquerytr`，每欄提供輸入框與運算子切換按鈕（`%` → `%%` → `=` → `>=` → `<=` → `!=` → `in` 循環），按 refresh 按鈕執行 `autoQuery` |

### 欄位隱藏持久化

- `columnHidable: true` 時，欄位標題及 label 均可透過 popover 按鈕隱藏
- 隱藏設定存入 `localStorage`，key 格式為 `datagrid_{id}_hcolumns_{user}_{pathname}`
- 工具列右側顯示眼睛圖示（`.show-column`），點擊可恢復已隱藏的欄位

### 內建編輯器類型

`$.fn.datagrid.defaults.editors` 定義了以下行內編輯器，用於 `beginEdit` 時依 Column 的 `editor.type` 建立：

`label`、`textbox`、`password`、`textarea`（彈出 modal 編輯）、`htmleditor`（彈出 UMEditor modal）、`combobox`、`refval`、`autocomplete`、`numberbox`、`datebox`、`datetimebox`、`timebox`、`dateselect`、`switch`、`options`、`fileupload`、`creator`、`updater`、`submenu`

每個編輯器實作四個方法：`init(container, options)`、`getValue(target)`、`setValue(target, value)`、`readonly(target, value)`。

### ChatUX 整合

init 階段會偵測 URL 中的 `c=` 參數與 `sessionStorage` 中的聊天指令，自動轉換為查詢條件或操作指令：
- 「新增」→ 設定 `loadAction = 'insert_row'`
- 「更改」→ 設定 `loadAction = 'edit_row'`
- 「刪除」→ 設定 `loadAction = 'delete_row'`
- 「印表」/「統計」→ 尋找匯出按鈕並設定對應 `loadAction`

未能直接匹配的查詢欄位名稱會透過 AJAX 呼叫 `chatUXGetColumn` 以 AI 方式模糊比對。

### 其他前端特有行為

- **XSS 防護**：`validateXss` 為 true 時，顯示值會經 `$.validateScript` 過濾；編輯時 `validateRow` 會呼叫 `$.xssValidate` 檢查
- **Record Lock**：`recordLock` 為 true 時，`edit_row` 和 `delete_row` 在進入編輯前會透過 `$.addLock` 向伺服器請求記錄鎖定
- **Creator/Updater 自動填入**：`updateRow` 時自動以 `sessionStorage.clientInfo.user` 及當前時間填入 creator/updater 欄位
- **CarryOn 預設值**：新增列時，若欄位設定 `carryOn: true`，會自動帶入上一筆編輯過的列的同欄位值（存於 `jq.data('lastRow')`）
- **多列欄位（trLength > 1）**：當 Column 設定 `alignTop` 指向另一欄位時，該欄位會成為子列，一筆資料渲染為多個 `<tr>`。此時自動停用 `table-hover` 和 `table-striped`
- **離線模式**：Webview 環境下偵測 `viewOffline` 按鈕並呼叫 `getOfflineNum` 顯示離線資料數量
- **Drill-down**：init 時檢查 URL 加密參數 `targetRemoteName`，若與當前 `remoteName` 相符則套用 drill-down 的 `whereItems` 和 `loadAction`

## 備註

- DataGrid 是 RWD 前端最常用的元件，幾乎所有模組都會使用。
- Column 渲染時會自動整合 `Default`（預設值）、`Validate`（驗證）、`AutoSeq`（自動序號）等元件的設定。
- Column 的 `Format` 設為 `map` 時，會自動加載 Map 編輯器類型。
- Column 的 `Formatter` 若未設定，會根據 `Format` 或 `Relation` 自動指派預設格式化函式（`formatValue` 或 `formatDisplay`）。
- `[MergeKey("id")]` 標記的屬性表示在方案合併時以 id 為鍵進行合併。
- `[Security]` 標記的屬性（如 ViewCommandVisible、EditCommandVisible、DeleteCommandVisible、Column.Hidden、ToolItem.Hidden）可在安全設定中依使用者/群組控制可見性。
- `[Localization]` 標記的屬性支援多語系翻譯。
