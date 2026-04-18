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

## JavaScript 實戰範例（討論區彙整）

### 1. 基本取值 / 設值

```javascript
// 取值
var v = $('#dfMaster_部門').combobox('getValue');

// 設值（會自動觸發 change）
$('#dfMaster_部門').combobox('setValue', 'HR');

// 多選設值：用 multipleSeparator 分隔字串（#480983）
$('#dfMaster_主管們').combobox('setValue', 'A主管,B主管,C主管');

// 取多選值：回傳以 separator 合併的字串
var managers = $('#dfMaster_主管們').combobox('getValue');   // "A主管,B主管,C主管"
var arr = managers.split(',');
```

### 2. 手動 loadData 灌入選項（#481091 / #481858 / #481039）

若不想綁 InfoCommand，想 JS 自行塞選項：

```javascript
$('#dfMaster_類型').combobox('loadData', [
    { value: 'A', text: '類型 A' },
    { value: 'B', text: '類型 B' },
    { value: 'C', text: '類型 C' }
]);
```

要用自訂 valueField / textField：
```javascript
$('#dfMaster_項目').combobox('options').valueField = 'id';
$('#dfMaster_項目').combobox('options').textField  = 'name';
$('#dfMaster_項目').combobox('loadData', [
    { id: 1, name: '蘋果' },
    { id: 2, name: '香蕉' }
]);
```

### 3. `onSelect` 取得**非 valueField 的其他欄位**（#480398）

`onSelect` 事件只給 value，若要取 row 其他欄位，用 `$.getDisplayRow`：

```javascript
function onPartSelect(value) {
    if (!value) return;

    var opts = $('#dfMaster_零件').combobox('options');
    $.getDisplayRow(opts, value, function(row) {
        if (row) {
            // row 包含完整欄位
            $('#dfMaster_單價').val(row.UNIT_PRICE);
            $('#dfMaster_庫存').val(row.STOCK);
            $('#dfMaster_供應商').val(row.SUPPLIER);
        }
    }, false);   // false = 同步取得
}
```

參考討論區 [#479232](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=479232) 的完整 pattern。

### 4. 多階下拉連動（#480501 / #481852 / #481389）

**情境**：選了部門（combobox1）→ 員工（combobox2）只顯示該部門的人。

**官方建議做法 — 用 `whereItems` 屬性，不用自己寫 setWhere**（[#480501](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480501) Roland 原話）：

```json
// combobox2 (員工) 的設計時 whereItems
{
    "type": "combobox",
    "id": "員工",
    "remoteName": "員工資料",
    "valueField": "EMP_ID",
    "textField": "EMP_NAME",
    "whereItems": [
        {
            "targetField": "DEPT_ID",         // 員工資料表的欄位
            "operator": "=",
            "sourceField": "部門"             // 從同表單的「部門」欄位取值
        }
    ]
}
```

**框架會自動**：
- combobox1（部門）change → combobox2 偵測到 sourceField 改變 → 自動重 load
- 不用寫 JS、不用手動 setWhere

**若需要 JS 層處理**（例如要做轉換）：

```javascript
function dept_onSelect(value) {
    if (!value) {
        $('#dfMaster_員工').combobox('setWhere', '1=0');
        return;
    }
    $('#dfMaster_員工').combobox('setWhere', "DEPT_ID = '" + value + "'");
    $('#dfMaster_員工').combobox('setValue', '');   // 清空舊選
}
```

### 5. Combobox ↔ RefVal 連動（#481389）

基本原理相同，`whereItems` 裡 `sourceField` 可以指到同表單 combobox 或 refval 的欄位：

```json
// 員工 RefVal 設 whereItems
{
    "type": "refval",
    "whereItems": [
        { "targetField": "DEPT_ID", "operator": "=", "sourceField": "部門" }
    ]
}
```

### 6. 動態切換 `remoteName`（#482342 多 Schema）

```javascript
function onSchemaChange(schema) {
    if (schema === 'PROD') {
        $('#dfMaster_品號').combobox('options').remoteName = '出貨單.refItemProd';
    } else {
        $('#dfMaster_品號').combobox('options').remoteName = '出貨單.refItemTest';
    }
    $('#dfMaster_品號').combobox('load');   // 重新載入
}
```

### 7. 動態切換 `textField`（多語系 #482062）

```javascript
function switchLocale() {
    var locale = $.getVariableValue('locale');   // 取系統語系
    var opts = $('#dfMaster_類型').combobox('options');

    if (locale === 'en-US')      opts.textField = 'TEXT_EN';
    else if (locale === 'zh-CN') opts.textField = 'TEXT_CN';
    else                         opts.textField = 'TEXT_TW';

    $('#dfMaster_類型').combobox('load');
}
```

> 建議 InfoCommand 裡的 SQL 已經把各語系欄位準備好（`SELECT VALUE, TEXT_TW, TEXT_EN, TEXT_CN FROM ...`），動態切 textField 即可。

### 8. 取得所有選項項目（#478578）

```javascript
// 以 jQuery 取 DOM 的 <option>
$('#dfMaster_類型 option').each(function() {
    console.log($(this).val(), $(this).text());
});

// 或從 combobox options 的 data 陣列
var opts = $('#dfMaster_類型').combobox('options');
console.log(opts.data);
```

### 9. 在 DataGrid 編輯中動態改 Combobox 選項（#481058）

```javascript
// dgMaster_onShowEditor 觸發時
function dgMaster_onShowEditor(index, field, editor) {
    if (field === '品號') {
        // editor.target 是該 cell 的 <select>
        $(editor.target).combobox('loadData', newOptions);
    }
}
```

### 10. call ServerMethod 塞值要用 `callSyncMethod`（#473028）

**❌ 錯誤**（非同步，setValue 時 result 還沒回來）：
```javascript
function dept_onSelect(value) {
    $.callMethod('模組', 'GetDefaultEmployee', { dept: value }, function(emp) {
        $('#dfMaster_員工').combobox('setValue', emp);
    });
    // 若是在 onSelect 最後 setValue，可能拿到空值
}
```

**✅ 正確**（同步）：
```javascript
function dept_onSelect(value) {
    var emp = $.callSyncMethod('模組', 'GetDefaultEmployee', { dept: value });
    $('#dfMaster_員工').combobox('setValue', emp);
}
```

## 常見陷阱與限制

| 陷阱 / 限制 | 說明 | 對策 | 來源 |
|------------|------|------|------|
| **檢視狀態只顯示 value 不顯示 text** | Combobox 的 textField 只在編輯狀態生效 | DataGrid 用 `Relation` 屬性；DataForm 用 formatter 轉文字 | [#481559](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481559) |
| **SP 不能當 Combobox 資料來源** | Stored Procedure 缺查詢欄位傳參機制 | 只能用一般 Text SQL InfoCommand；SP 僅限 DataGrid | [#480584](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480584) / [#482342](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482342) |
| **不支援「自行輸入值」** | 只能從清單選 | 改用 **Autocomplete** 元件（[#482586](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482586) / [#477368](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477368)） | |
| **`onSelect` 只給 value 拿不到其他欄位** | 事件 callback 參數只有一個 value | 用 `$.getDisplayRow(opts, value, callback)` 查 row | [#480398](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480398) |
| **callMethod 塞值拿到空** | 非同步 callback 時機晚於 setValue | 改用 **`$.callSyncMethod`** | [#473028](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473028) |
| **多階連動自己寫 setWhere 沒效果** | 沒正確使用 whereItems 屬性 | 用 `whereItems` 的 `sourceField` 指到父欄位，框架自動處理 | [#480501](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480501) |
| **Multiple + loadData 載入後選不到最後一個** | selectpicker 未 refresh | loadData 後手動 `$('#id').selectpicker('refresh')` | [#474218](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474218) / [#475170](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=475170) / [#474856](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474856) |
| **多語系只能切 textField，不能多 InfoCommand** | Combobox 一個 remoteName 綁死 | 在 InfoCommand 準備各語系欄位，JS 動態切 textField | [#482062](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482062) |
| **部署 IIS 後下拉選單消失** | 靜態檔路徑或 JS 未正確載入 | 檢查 IIS 虛擬目錄 / MIME 設定 | [#480632](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480632) |
| **編輯狀態 setValue 沒反應** | editor 未渲染完就 setValue | 在 `onShowEditor` 用 `setTimeout(..., 0)` 延一個 tick | [#473329](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473329) |
| **查詢介面 combobox loadData 後每次查詢被覆蓋** | Query 面板 reload 時重載 | 在 `onQuery` 先存全域變數，再還原 | [#481858](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481858) |
| **fromSysParameters + InfoCommand 可混用嗎** | 設計上**擇一**即可 | `fromSysParameters=true` 會自動蓋 remoteName 為 `SystemTable.sysParas` | 原始碼 L9559 |

## 備註

- 內部類別 `Item` 包含 `Value` 和 `Text` 兩個屬性。
- `WhereItems` 共用 `Refval.WhereItem` 類別，支援動態篩選條件。
- **placeholder「請選擇」預設已內建**（原始碼 `loadData` 會插入隱藏提示項，多選模式顯示 `$.fn.locale.nothingselected`）—[#481616](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481616)
- **`onSelect` 事件簽名**：`function(value)` — 只給 value（不含整 row）
- **檢視（view）狀態下 combobox 不會渲染文字**，只顯示原始 value。要在檢視下看到文字：
  - DataGrid：用 column 的 `Relation` 屬性對應 InfoCommand 自動 resolve
  - DataForm：用 formatter 函式 或 onLoad 時手動 resolve
