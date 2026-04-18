# Tree

> `EEPRWDTools.Core/Controls/Tree.cs` — 94 行
> 繼承：`RWDControl` → `Component`

## 用途

**樹狀結構元件**（Tree）。

Tree 用來以階層樹狀結構呈現具有父子關係的資料。支援指定 ID 欄位、父節點欄位、顯示文字欄位，可連動目標元件（如 DataGrid、Schedule），並支援節點編輯、新增、及篩選條件。

## JSON 設定範例

```json
{
  "type": "tree",
  "id": "treeDept",
  "remoteName": "cmdDept",
  "idField": "DEPT_ID",
  "parentField": "PARENT_ID",
  "textField": "DEPT_NAME",
  "targetObject": "gridEmployee",
  "title": "部門",
  "levels": 2,
  "editable": false,
  "allowAdd": false,
  "showBorder": false,
  "whereItems": [
    { "targetField": "DEPT_ID", "operator": "=", "sourceField": "DEPT_ID" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源 RemoteName |
| **idField** | string | 欄位選擇器 `[ColumnEditor]` | — | 節點 ID 欄位 |
| **parentField** | string | 欄位選擇器 `[ColumnEditor]` | — | 父節點欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 節點顯示文字欄位 |
| **targetObject** | string | 元件選擇器 `[ControlEditor]`（datagrid, schedule） | — | 連動目標元件 ID |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | 空集合 | 連動篩選條件 |
| **title** | string | 文字（多語系） `[Localization]` | — | 樹狀結構標題 |
| **editable** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否可編輯節點 |
| **allowAdd** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否允許新增節點 |
| **editForm** | string | 元件選擇器 `[ControlEditor]`（dataform） | — | 編輯表單元件 ID |
| **levels** | int | 數字框 `[NumberboxEditor]`（min=1, 預設=2） | 2 | 樹狀展開層數 |
| **nodeIcon** | string | 圖示選擇器 `[IconRWDEditor]` | — | 節點圖示 CSS class |
| **showBorder** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否顯示邊框 |
| **borderColor** | bool | 顏色選擇器 `[ColorEditor]` | — | 邊框顏色 |
| **width** | int? | 數字框 `[NumberboxEditor]` | — | 元件寬度（px） |

### 事件屬性

| 屬性 | 參數 | 說明 |
|------|------|------|
| **onBeforeLoad** | `(param)` | 載入前事件，可修改查詢參數 |
| **onRenderNode** | `(row)` | 節點渲染事件，可自訂節點呈現 |
| **onNodeSelected** | `(event, node)` | 節點選取事件 |
| **onUpdate** | `(row)` → `return true;` | 更新前事件，回傳 false 可取消 |

### WhereItem 子類別

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **targetField** | string | 欄位選擇器 `[ColumnEditor]`（target） | — | 目標欄位 |
| **operator** | string | 運算子選擇器 `[OperatorEditor]` | `"="` | 比較運算子 |
| **sourceField** | string | 欄位選擇器 `[ColumnEditor]` | — | 來源欄位 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 13699–13928 行（`$.fn.tree`）

### 渲染結構

`init` 流程：
1. 若有 `title`，在元素前插入 `<p class="datagrid-title">`。
2. 設定元素寬度（`width`）。
3. 呼叫 `createHeader` → `load` → `bindEvent`。

`createHeader`：若 `allowAdd && editable`，在元素前插入工具列 `<div class="datagrid-toolitem tree-header">`，含「新增」按鈕。

### 公開 API 方法

| 方法 | 說明 |
|------|------|
| `$el.tree('options')` | 取得元件選項物件 |
| `$el.tree('load')` | 透過 `$.loadData` 從後端載入資料並呼叫 `loadData` 渲染 |
| `$el.tree('loadData', rows)` | 將扁平資料轉為樹狀結構並初始化 `treeview` 外掛 |
| `$el.tree('getSelected')` | 取得目前選取的節點（回傳第一個 selected node 或 null） |
| `$el.tree('insert_row')` | 開啟 `editForm` 進行新增（status='inserted'） |
| `$el.tree('delete_row')` | 刪除選取節點（確認後呼叫 `$.updateData`，重新載入樹） |
| `$el.tree('setWhere', where)` | 設定篩選條件並自動重新 `load` |
| `$el.tree('renderNode', target, props)` | 更新指定節點的 row 資料及顯示文字 |

### 關鍵行為

**1. 資料載入與樹狀轉換（load / loadData）**
- `load` 以 `$.loadData` 取得遠端資料（含 keys），支援 `whereStr` 與 `onBeforeLoad` 回呼。
- `loadData` 內部的 `getNodeData` 遞迴函式將扁平 rows 依 `parentField` / `idField` 組成巢狀結構。
- 根節點的判定：`parentField` 值為空字串、null 或 undefined 的 row 視為根節點（與空字串比對）。
- 每個節點物件結構：`{ text, row, nodes }` — `nodes` 為子節點陣列（無子節點時為 null）。
- 若有定義 `onRenderNode`，節點 `text` 會改用該回呼的回傳值。

**2. treeview 外掛整合**
- 使用 Bootstrap `treeview` 外掛渲染（`$(this).treeview({...})`）。
- 傳入選項包含：`levels`（展開層數）、`nodeIcon`（glyphicon class）、`borderColor`、`showBorder`、`data`（樹狀資料）。
- 視覺樣式（`backColor`、`color`、`selectedBackColor`、`selectedColor`、`onhoverColor`）皆設為空字串，使用 CSS 控制。

**3. 節點選取與目標連動（onNodeSelected）**
- 選取節點時，若有 `targetObject` 與 `whereItems`，會將 whereItems 的 `sourceField` 對應到節點 row 的值，組成篩選條件。
- 目標元件支援 `datagrid` 與 `schedule` 兩種類型，透過 `setWhere` 方法傳遞篩選條件。
- 若有 `editForm`：觸發 `onUpdate` 回呼，在 Modal footer 動態插入「刪除」按鈕（僅 editable 時），以 `form('open')` 開啟表單（status 為 `updated` 或 `view`）。
- 最後觸發自訂的 `onNodeSelected` 事件回呼。

**4. 新增與刪除節點**
- `insert_row`：透過 `editForm` 的 `form('getDefaultValues')` 取得預設值，以 `status='inserted'` 開啟表單。
- `delete_row`：顯示確認對話框後，呼叫 `$.updateData` 將選取節點的 row 放入 `deleted` 陣列送出。成功後關閉 Modal 並重新 `load` 整棵樹。

**5. renderNode 動態更新**
- 透過 `data-nodeid` 找到 treeview 內部節點物件，合併 `props` 到 `node.row`，重新計算 `text`（支援 `onRenderNode`）。

## 實戰範例：動態篩選 Tree 資料

> 來源：[#482305](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482305)（新鷹精器，BOM 上下階查詢）

### `onBeforeLoad` 的 param 物件

從原始碼（L13758-13764）：

```javascript
var param = {
    total: true,
    whereStr: opts.whereStr       // ← 只有 whereStr，沒有 whereItems
};
if (opts.onBeforeLoad) {
    opts.onBeforeLoad.call(this, param);
}
$.loadData(opts.remoteName, param, ...)
```

**重點**：Tree 的 `onBeforeLoad` 只能改 **`param.whereStr`**（字串型）。不能像 DataGrid 用 `whereItems` 陣列。

### 首次載入不顯示資料（保留手動查詢後才載入）

```javascript
var isFirstTime = true;

function Tree1_onBeforeLoad(param) {
    if (isFirstTime) {
        param.whereStr = '1=0';   // 首次：SQL 永遠為 false → 無資料
        isFirstTime = false;
        return;
    }
    // 後續：維持 setWhere 設好的 whereStr
}
```

### ⚠️ 不要在 `onBeforeLoad` 內呼叫 `setWhere`（無限迴圈）

**錯誤寫法**：
```javascript
function Tree1_onBeforeLoad(param) {
    $('#Tree1').tree('setWhere', '1=0');   // ❌ 會無限迴圈
}
```

**原因**：`setWhere` 內部會呼叫 `load()`，`load` 又觸發 `onBeforeLoad` → 再 `setWhere` → 無限迴圈（瀏覽器可能當機）。

**正解**：直接改 `param.whereStr` 即可，不要經 `setWhere`。

### BOM 上下階樹狀過濾的完整 pattern

**情境**：DataGrid 查詢某個文件編號 → Tree 只顯示該文件的所有上下階（祖先+子孫），其他無關文件不顯示。

**挑戰**：單純 `WHERE doc_no = 'C'` 只會得到單一節點 C，但 Tree 組樹狀時找不到 C 的父節點 A、B，於是整棵樹消失（即使樹含 A-B-C-D-E，過濾後因為父節點缺失而顯示空）。

**解法**：用 **recursive CTE** 找出所有相關節點 ID，再用 `IN (...)` 過濾。

#### Server C# Method（用 SQL Server recursive CTE）

```csharp
public object QueryRelatedData(dynamic objParam)
{
    string docNo = Convert.ToString(objParam["docNo"]);

    if (string.IsNullOrWhiteSpace(docNo))
        return new { where = "1=1" };   // 空值 → 顯示全部（或回傳無條件）

    // 單引號 escape，防 SQL Injection
    string docNoSql = docNo.Replace("'", "''");

    string sql = @"
        ;WITH StartNode AS (
            SELECT OID FROM Documents WHERE doc_no = '" + docNoSql + @"'
        ),
        UpCTE AS (
            -- 往上找祖先
            SELECT OID FROM StartNode
            UNION ALL
            SELECT dl.upperDocOID
            FROM UpCTE u
            JOIN Doc_Level dl ON dl.lowerDocOID = u.OID
        ),
        DownCTE AS (
            -- 往下找子孫
            SELECT OID FROM StartNode
            UNION ALL
            SELECT dl.lowerDocOID
            FROM DownCTE d
            JOIN Doc_Level dl ON dl.upperDocOID = d.OID
        ),
        AllNodes AS (
            SELECT OID FROM UpCTE
            UNION
            SELECT OID FROM DownCTE
        )
        SELECT OID FROM AllNodes
        OPTION (MAXRECURSION 32767);";

    DataTable dt = ExecuteDataTable(sql);

    if (dt == null || dt.Rows.Count == 0)
        return new { where = "1=0" };

    // 收集 OID，組 IN 清單（每個值仍 escape 單引號）
    var oids = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
    foreach (DataRow r in dt.Rows)
    {
        var oid = Convert.ToString(r["OID"]);
        if (!string.IsNullOrWhiteSpace(oid))
            oids.Add(oid);
    }

    string inList = string.Join(",", oids.Select(x => "'" + x.Replace("'", "''") + "'"));
    return new { where = $"OID IN ({inList})" };
}
```

#### 前端 RWD JS

```javascript
var treeWhereStr = '';
var isFirstTime = true;

function Tree1_onBeforeLoad(param) {
    if (isFirstTime) {
        param.whereStr = '1=0';    // 首次不載入
        isFirstTime = false;
        return;
    }
    param.whereStr = treeWhereStr;
}

function dgMaster_onQuery(whereItems) {
    var docNo = whereItems[0].value;

    // 呼叫 Server Method 找所有相關節點
    var res = $.callSyncMethod('文件模組', 'QueryRelatedData', { docNo: docNo });
    treeWhereStr = (res && res.where) ? res.where : '1=0';

    // 觸發 Tree 重新載入（會走 onBeforeLoad 套用新 whereStr）
    $('#Tree1').tree('load');

    // （可選）自動收合
    setTimeout(function() {
        $('#Tree1 .expand-icon.glyphicon.glyphicon-minus').each(function() {
            $(this).click();
        });
    }, 100);

    return true;
}
```

### 為什麼需要 Server Method 而不是純 SQL

- **setWhere 的 whereStr 直接接進 InfoCommand 的 WHERE 後面**，但 InfoCommand 的 SQL 通常是簡單的 `SELECT * FROM Documents`，不會支援 recursive CTE
- 要做上下階展開，**必須先把所有相關節點的 ID 算出來**（一次 SQL recursion），然後用 `IN (id1, id2, ...)` 當 setWhere 條件

## 常見陷阱

| 陷阱 | 說明 | 對策 |
|------|------|------|
| **在 `onBeforeLoad` 呼叫 `setWhere` 造成無限迴圈** | `setWhere → load → onBeforeLoad → setWhere...` | 直接改 `param.whereStr` 即可，不要呼叫 setWhere |
| **過濾單一節點導致整棵樹空** | 父節點不在結果內，樹狀組合找不到根 | 用 recursive CTE 一併帶回所有祖先、子孫 ID |
| **Tree 沒有 `whereItems` 選項** | `onBeforeLoad` 的 param 只有 `whereStr` | 組字串 SQL；多條件自行 AND / OR 拼接 |
| **Tree 資料量大 → 頁面 lag** | 數千筆節點全部展開 | 設 CSS `max-height + overflow-y: auto`、控制 `levels` |
| **setWhere 是非同步** | `setWhere` 內的 `load` 是 AJAX | 後續依賴資料的動作用 `setTimeout` 或等事件 |

### 顯示範圍控制（CSS）

若只是**畫面拖拉太長**，不必從資料層過濾，改用 CSS 即可：

```html
<!-- 用 Literal 元件放入 -->
<style>
#Tree1 {
    max-height: 500px;      /* 自訂高度 */
    overflow-y: auto;       /* 超出顯示垂直捲軸 */
}
</style>
```

## 備註

- 渲染時輸出 `<div class="bootstrap-tree">`。
- WhereItems 定義樹狀節點選取時傳遞給 TargetObject 的篩選條件，實現主從連動。
- `BorderColor` 屬性類型宣告為 `bool`（可能為原始碼瑕疵，設計介面標記為 `[ColorEditor]`）。
- **`onBeforeLoad` 只能改 `whereStr`（字串），無 `whereItems` 選項**（與 DataGrid 不同）。
- `setWhere` 傳 Array 會被忽略（參見 L13898-13903），實際上只有字串生效。
