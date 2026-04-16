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

## 備註

- 渲染時輸出 `<div class="bootstrap-tree">`。
- WhereItems 定義樹狀節點選取時傳遞給 TargetObject 的篩選條件，實現主從連動。
- `BorderColor` 屬性類型宣告為 `bool`（可能為原始碼瑕疵，設計介面標記為 `[ColorEditor]`）。
