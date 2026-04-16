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

## 備註

- DataGrid 是 RWD 前端最常用的元件，幾乎所有模組都會使用。
- Column 渲染時會自動整合 `Default`（預設值）、`Validate`（驗證）、`AutoSeq`（自動序號）等元件的設定。
- Column 的 `Format` 設為 `map` 時，會自動加載 Map 編輯器類型。
- Column 的 `Formatter` 若未設定，會根據 `Format` 或 `Relation` 自動指派預設格式化函式（`formatValue` 或 `formatDisplay`）。
- `[MergeKey("id")]` 標記的屬性表示在方案合併時以 id 為鍵進行合併。
- `[Security]` 標記的屬性（如 ViewCommandVisible、EditCommandVisible、DeleteCommandVisible、Column.Hidden、ToolItem.Hidden）可在安全設定中依使用者/群組控制可見性。
- `[Localization]` 標記的屬性支援多語系翻譯。
