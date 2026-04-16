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

## 備註

- 渲染時輸出 `<div class="bootstrap-tree">`。
- WhereItems 定義樹狀節點選取時傳遞給 TargetObject 的篩選條件，實現主從連動。
- `BorderColor` 屬性類型宣告為 `bool`（可能為原始碼瑕疵，設計介面標記為 `[ColorEditor]`）。
