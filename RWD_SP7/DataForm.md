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

## JSON 設定範例

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

## 渲染流程

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

## 備註

- DataForm 的 `Mode` 決定渲染方式：`Dialog` 和 `Tab` 渲染為 Bootstrap Modal，`Panel` 渲染為可摺疊的 Panel。
- `HorizontalColumnsCount` 控制 Bootstrap Grid 的欄位寬度計算：`colSpan = 12 / horizontalColumnsCount * span - labelSpan`。當 `HorizontalColumnsCount >= 4` 時，label 佔 1 格；否則佔 2 格。
- 表單渲染時會自動整合同一容器內的 `Default`、`Validate`、`AutoSeq` 元件，將預設值和驗證規則注入到對應的 Column。
- `DuplicateCheck` 啟用時，前端會在送出前檢查是否有重複資料。
- `CloseProtect` 啟用時，使用者關閉視窗會收到未儲存變更提示。
- `FormCls` 為直接 field（非 property），可在程式碼中設定額外的 CSS class。
- ToolItem 的 `Render()` 方法會判斷 `IconCls` 前綴：以 `fa` 開頭使用 Font Awesome，否則使用 Glyphicon。
- DataPanel 子控件的 `PId` 會在 `Render()` 時被自動設定為 DataForm 的 Id。
