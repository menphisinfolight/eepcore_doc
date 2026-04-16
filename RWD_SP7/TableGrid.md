# TableGrid

> `EEPRWDTools.Core/Controls/TableGrid.cs` — 379 行
> 繼承：`RWDControl` → `Component`

## 用途

**表格式資料網格元件**（Table Grid）。

TableGrid 是 DataGrid 的簡化版本，提供類似的資料瀏覽與編輯功能，但以 Bootstrap Table 樣式呈現（`bootstrap-tablegrid`），而非 DataGrid 的 EasyUI datagrid。設計師在模組 JSON 中以 `"type": "tablegrid"` 宣告。

TableGrid 內部的 `Render()` 方法實際上會建立一個 `DataGrid` 實例，將自身屬性透過反射 `CopyProperty()` 複製過去，再呼叫 `DataGrid.Render()` 以 `"bootstrap-tablegrid"` 範本輸出 HTML。因此 TableGrid 本質上是 DataGrid 的**輕量包裝器**，共用底層渲染邏輯。

## JSON 設定範例

```json
{
  "type": "tablegrid",
  "id": "gridOrder",
  "remoteName": "orderCommand",
  "editForm": "formOrder",
  "columns": [
    { "field": "ORDER_NO", "title": "訂單編號", "cWidth": 120 },
    { "field": "ORDER_DATE", "title": "訂單日期", "format": "yyyy/MM/dd", "cWidth": 100 },
    { "field": "AMOUNT", "title": "金額", "alignment": "right", "format": "N0", "total": "Sum" }
  ],
  "toolItems": [
    { "text": "Add", "iconCls": "glyphicon-plus", "onclick": "insert_row" },
    { "text": "Delete", "iconCls": "glyphicon-remove", "onclick": "delete_row" }
  ],
  "pagination": true,
  "pageSize": 20,
  "height": 400,
  "bordered": true,
  "hover": true
}
```

## 設計介面屬性

| 屬性 | 類型 | 預設值 | 設計介面 | 說明 |
|------|------|--------|----------|------|
| **RemoteName** | string | — | 命令選擇器 `[CommandEditor]` | 綁定的伺服器端資料命令（InfoCommand） |
| **Columns** | List\<Column\> | [] | 集合編輯器 `[CollectionEditor]` | 欄位定義集合（詳見 Column 內部類別） |
| **WhereStr** | string | — | 文字 | 靜態 WHERE 條件字串 |
| **AutoApply** | bool | `true` | 勾選 `[CheckboxEditor]` | 是否自動送出變更 |
| **ConfirmDelete** | bool | `true` | 勾選 `[CheckboxEditor]` | 刪除前是否跳確認對話框 |
| **DuplicateCheck** | bool | `true` | 勾選 `[CheckboxEditor]` | 是否檢查重複資料 |
| **ToolItems** | List\<ToolItem\> | [] | 集合編輯器 `[CollectionEditor]` | 自訂工具列按鈕（詳見 ToolItem） |
| **ToolItemPosition** | ToolItemPosition | `Top` | 下拉 | 工具列位置：`Top` / `Bottom` |
| **ViewCommandVisible** | bool | `false` | 安全控制 `[Security]` | 檢視按鈕是否可見 |
| **EditCommandVisible** | bool | `false` | 安全控制 `[Security]` | 編輯按鈕是否可見 |
| **DeleteCommandVisible** | bool | `false` | 安全控制 `[Security]` | 刪除按鈕是否可見 |
| **ColumnHidable** | bool | `false` | — | 是否允許使用者隱藏欄位 |
| **EditForm** | string | — | 控制項選擇器 `[ControlEditor]` | 編輯用的 DataForm 元件 ID |
| **ParentObject** | string | — | 控制項選擇器 `[ControlEditor]` | 父層物件（DataGrid / DataForm） |
| **TargetObject** | string | — | 控制項選擇器 `[ControlEditor]` | 目標物件（DataGrid / DataForm） |
| **WhereItems** | List\<WhereItem\> | [] | 集合編輯器 `[CollectionEditor]` | 查詢條件集合（與 DataGrid 共用） |
| **TotalMode** | TotalMode | `Page` | 下拉 | 合計模式：`Page`（當頁）/ `All`（全部） |
| **QueryColumns** | List\<QueryColumn\> | [] | 集合編輯器 `[CollectionEditor]` | 查詢面板欄位定義 |
| **QueryMode** | QueryMode | `Dialog` | 下拉 | 查詢面板模式：`Dialog` / `Panel` |
| **QueryTitle** | string | — | 文字（多語系 `[Localization]`） | 查詢面板標題 |
| **QueryColumnsCount** | int | `2` | 數字框 `[NumberboxEditor]` | 查詢面板每行欄位數 |
| **ShowCheckbox** | bool | `false` | — | 是否顯示勾選框 |
| **RecordLock** | bool | `false` | — | 是否啟用記錄鎖定 |
| **Pagination** | bool | `true` | 勾選 `[CheckboxEditor]` | 是否啟用分頁 |
| **PageJump** | bool | `false` | 勾選 `[CheckboxEditor]` | 是否顯示頁碼跳轉 |
| **Rownumbers** | bool | `false` | — | 是否顯示行號 |
| **ReportName** | string | — | 選單選擇器 `[MenuEditor]` | 報表名稱 |
| **PageSize** | int | `10` | 數字框 `[NumberboxEditor]` | 每頁筆數 |
| **PageList** | JArray | [] | 陣列編輯器 `[ArrayEditor]` | 可選的每頁筆數清單 |
| **Title** | string | — | 文字（多語系 `[Localization]`） | 表格標題 |
| **Height** | int? | `400` | 數字框 `[NumberboxEditor]` | 表格高度（px） |
| **FixedColumns** | int | `0` | 數字框 `[NumberboxEditor]` | 凍結欄位數（從左起） |
| **RowStyler** | string | — | 腳本編輯器 `[ScriptEditor]` | 行樣式函數 `(index, row)` |
| **Bordered** | bool | `true` | 勾選 `[CheckboxEditor]` | 是否顯示框線 |
| **Hover** | bool | `true` | 勾選 `[CheckboxEditor]` | 是否顯示 hover 效果 |
| **Striped** | bool | `false` | — | 是否顯示斑馬紋 |
| **Condensed** | bool | `false` | — | 是否使用緊湊模式 |
| **Xsblock** | bool | `true` | 勾選 `[CheckboxEditor]` | 手機版是否佔滿寬度 |
| **Simple** | bool | `false` | — | 是否使用簡易模式 |

## Column 內部類別

欄位定義（`TableGrid.Column`），繼承 `RWDCollectionItem`。

| 屬性 | 類型 | 說明 |
|------|------|------|
| **Title** | string | 欄位標題（多語系 `[Localization]`） |
| **Field** | string | 對應的資料欄位名稱 `[ColumnEditor]` |
| **CWidth** | int? | 欄位寬度（px） |
| **Alignment** | string | 對齊方式：`left` / `center` / `right` |
| **Nowrap** | bool | 是否不換行 |
| **Hidden** | bool | 是否隱藏（安全控制 `[Security]`） |
| **Sortable** | bool | 是否可排序 |
| **Format** | string | 格式化類型，可選值：`N0`、`yyyy/MM/dd`、`logic`、`checkbox`、`image`、`file`、`barcode`、`qrcode`、`map`、`signature`、`drilldown`、`flowflag`、`creator`、`updater` |
| **Formatter** | string | 自訂格式化函數 `(value, row, index) => return value;` |
| **Total** | Total | 合計方式（如 Sum、Count 等） |
| **Relation** | Relation | 關聯設定（顯示關聯資料的文字而非代碼） |

### Column.Render() 邏輯

當 `Formatter` 為空時，會依條件自動指派：
- 有 `Format` 設定 → 使用 `$.fn.TableGrid.methods.formatValue`
- 有 `Relation.RemoteName` 設定 → 使用 `$.fn.TableGrid.methods.formatDisplay`

## ToolItem 定義

TableGrid 的工具列按鈕（`TableGrid.ToolItem`），實作 `IToolItem` 介面。

### 內建工具列按鈕

| Text | 圖示 | Onclick | 說明 |
|------|------|---------|------|
| Add | glyphicon-plus | `insert_row` | 新增一筆資料 |
| Edit | glyphicon-edit | `edit_row` | 編輯選中資料 |
| Delete | glyphicon-remove | `delete_row` | 刪除選中資料 |
| Query | glyphicon-search | `openQuery` | 開啟查詢面板 |
| Copy | glyphicon-copy | `copy_row` | 複製選中資料 |
| Ok | glyphicon-ok | `endEdit` | 確認編輯 |
| Cancel | glyphicon-remove | `cancelEdit` | 取消編輯 |
| Import | glyphicon-import | `importExcel` | 匯入 Excel |
| Export | glyphicon-export | `export` | 匯出資料 |
| Print | glyphicon-print | `exportReport` | 列印報表 |
| UserDefined | glyphicon-print | `userDefined` | 使用者自訂 |
| UserPrint | glyphicon-print | `userPrint` | 使用者自訂列印 |

### ToolItem 屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| **Text** | string | 按鈕文字（多語系 `[Localization]`） |
| **HelpText** | string | 提示文字（以 popover 顯示，多語系） |
| **IconCls** | string | 圖示 CSS class `[IconRWDEditor]` |
| **IconAlign** | IconAlign | 圖示位置：`Left` / `Right` |
| **BtnCls** | string | 按鈕樣式 class `[BtnClsEditor]` |
| **Hidden** | bool | 是否隱藏（安全控制 `[Security]`） |
| **Onclick** | string | 點擊事件名稱 |

## 事件

| 事件 | 參數 | 預設回傳 | 說明 |
|------|------|----------|------|
| **OnBeforeLoad** | `(param)` | — | 資料載入前觸發，可修改查詢參數 |
| **OnLoad** | `(data)` | — | 資料載入完成後觸發 |
| **OnSelect** | `(index, row)` | — | 選取某筆資料時觸發 |
| **OnInsert** | `(row)` | `return true;` | 新增前觸發，回傳 `false` 可取消 |
| **OnUpdate** | `(row)` | `return true;` | 修改前觸發，回傳 `false` 可取消 |
| **OnDelete** | `(row)` | `return true;` | 刪除前觸發，回傳 `false` 可取消 |
| **OnQuery** | `(whereItems)` | `return true;` | 查詢前觸發，回傳 `false` 可取消 |
| **OnImportExcelSuccess** | — | — | Excel 匯入成功後觸發 |
| **OnShowEditor** | `(index, field, editor)` | `return editor;` | 顯示編輯器前觸發，可替換編輯器 |

## 與 DataGrid 的差異

| 比較項目 | TableGrid | DataGrid |
|----------|-----------|----------|
| **渲染範本** | `bootstrap-tablegrid`（Bootstrap Table） | `bootstrap-datagrid`（EasyUI 風格） |
| **Column 集合編輯器** | `"tablegrid", "column"` | `"datagrid", "column"` |
| **Height 預設值** | `400` | 無預設值（`null`） |
| **FixedColumns** | 支援（作為額外屬性傳入） | 不支援（透過 `FixColumnHeader` 固定表頭） |
| **Bordered / Hover / Striped / Condensed / Xsblock** | 有（Bootstrap Table 樣式控制） | 無 |
| **Simple** | 有 | 無 |
| **ValidateXss** | 無 | 有（預設 `true`） |
| **AlwaysClose** | 無 | 有 |
| **EditOnEnter** | 無 | 有（預設 `true`） |
| **FixColumnHeader** | 無 | 有 |
| **AUD_Detect** | 無 | 有 |
| **ShowColumnTitle** | 無（已註解） | 有（預設 `true`） |
| **AutoQueryColumn** | 無（已註解） | 有 |
| **CheckOnSelect** | 無（已註解） | 有 |
| **PageCount** | 無 | 有（預設 `true`） |
| **OnLoadError** | 無 | 有 |
| **OnDeleted** | 無 | 有 |
| **OnApplied** | 無 | 有 |
| **ReportType** | 無 | 有 |
| **Column.Editor** | 無 | 有（欄位內建編輯器） |

## 備註

- TableGrid 並非獨立渲染，而是在 `Render()` 中建立 `DataGrid` 實例，透過反射將同名屬性複製過去，再呼叫 `DataGrid.Render(html, "bootstrap-tablegrid", ...)`。因此 TableGrid 的前端行為與 DataGrid 基本一致，差別在 HTML 範本。
- `FixedColumns` 屬性透過 `properties` 參數以 `fixedColumns:N` 格式傳給 DataGrid，而非直接設為屬性。
- `CopyProperty()` 只複製有 `CanWrite`（具 setter）的屬性。`Columns`、`ToolItems`、`WhereItems`、`QueryColumns` 等集合屬性為唯讀（只有 getter），因此在 `Render()` 中以 foreach 逐一複製。
- TableGrid 的 `Column` 類別比 DataGrid 的 `Column` 精簡，少了 `Editor` 屬性，不支援欄位內直接編輯（inline editing），適合以 EditForm 方式編輯資料。
- 來源檔中有數個已註解的屬性（`ShowColumnTitle`、`AutoQueryColumn`、`CheckOnSelect`），表示這些功能曾考慮但未啟用。
- `using static EEPRWDTools.Core.Controls.DataGrid;` 靜態引用使 TableGrid 可直接使用 DataGrid 的內部類別，如 `WhereItem`、`QueryColumn`、`Relation`、`Total` 等。
