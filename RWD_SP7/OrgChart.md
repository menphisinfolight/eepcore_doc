# OrgChart

> `EEPRWDTools.Core/Controls/OrgChart.cs` — 106 行
> 繼承：`RWDControl` → `Component`

## 用途

**組織圖元件**（Org Chart）。

OrgChart 透過 RemoteName 綁定階層式資料，以 IdField/ParentField 建構父子關係樹，NameField 和 TitleField 分別顯示節點名稱與職稱。支援編輯模式、自訂工具按鈕、節點樣板、垂直層級控制。前端渲染為 `<div class="bootstrap-Orgchart">`。

## JSON 設定範例

```json
{
  "type": "orgchart",
  "id": "ocDept",
  "remoteName": "qryOrg",
  "idField": "EMP_ID",
  "parentField": "MANAGER_ID",
  "nameField": "EMP_NAME",
  "titleField": "TITLE",
  "editable": true,
  "editForm": "dfEmployee",
  "verticalLevel": 2,
  "toolItems": [
    {
      "text": "新增",
      "iconCls": "glyphicon-plus",
      "iconAlign": "Left",
      "btnCls": "btn-primary",
      "hidden": false,
      "onclick": "addNode()"
    }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **toolItems** | List\<ToolItem\> | 集合編輯器 `[CollectionEditor]` | [] | 工具列按鈕集合（見下方子屬性） |
| **idField** | string | 欄位選擇器 `[ColumnEditor]` | — | 節點 ID 欄位 |
| **parentField** | string | 欄位選擇器 `[ColumnEditor]` | — | 父節點 ID 欄位 |
| **nameField** | string | 欄位選擇器 `[ColumnEditor]` | — | 節點名稱欄位 |
| **titleField** | string | 欄位選擇器 `[ColumnEditor]` | — | 節點職稱欄位 |
| **editable** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否可編輯 |
| **editForm** | string | 元件選擇器 `[ControlEditor]`（限 dataform） | — | 編輯用表單元件 ID |
| **verticalLevel** | int | 數字框 `[NumberboxEditor]` | 1 | 垂直顯示的層級數 |

### 事件

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | 載入前事件（參數：param） |
| **onNodeClick** | string | 腳本編輯器 `[ScriptEditor]` | 節點點擊事件（參數：row） |
| **nodeTemplate** | string | 腳本編輯器 `[ScriptEditor]` | 節點樣板（參數：data） |

### ToolItem 子屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **text** | string | 多語系 `[Localization]` | — | 按鈕文字 |
| **iconCls** | string | 圖示選擇器 `[IconRWDEditor]` | — | Bootstrap glyphicon 圖示 class |
| **iconAlign** | IconAlign | — | — | 圖示位置（Left/Right） |
| **btnCls** | string | 按鈕樣式選擇器 `[BtnClsEditor]` | — | 按鈕 CSS class（如 btn-primary） |
| **hidden** | bool | — | — | 是否隱藏 |
| **onclick** | string | — | — | 點擊事件腳本 |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.Orgchart`（`bootstrap.infolight.js` 第 18957–19200 行）

### 初始化流程

1. 將工具列按鈕文字轉為多語系（`delete` → `remove`、`export` → `exports`）。
2. 綁定工具列 `.btn` 的 `click` 事件：優先呼叫 `$.fn.Orgchart.methods` 中的同名方法，否則以 `$.callFunction` 呼叫自訂函式。
3. 若設定 `onNodeClick`，綁定 `.node` 的點擊事件。
4. 呼叫 `load()` 載入資料。

### 主要方法

| 方法 | 說明 |
|------|------|
| `load()` | 以 `$.loadData(remoteName, ...)` 載入資料（含 `whereStr`），觸發 `onBeforeLoad` 可修改參數。載入後呼叫 `acceptChanges` 清空異動記錄，再呼叫 `loadData`。 |
| `loadData(rows)` | 以 `idField`/`parentField` 將扁平資料轉換為樹狀結構，呼叫第三方 `$.fn.orgchart` 渲染圖表。支援 `draggable`（依 `editable`）、`verticalLevel`、`nodeTemplate`。綁定 `nodedrop` 事件處理拖曳更新父節點。 |
| `insert_row()` | 取得選中節點（`.node.focused`）作為父節點，開啟 `editForm` 表單（`status: 'inserted'`）。 |
| `edit_row()` | 取得選中節點的資料，開啟 `editForm` 表單（`status: 'updated'`）。 |
| `delete_row()` | 確認刪除後，以 `$.updateData` 送出 `deleted` 陣列，成功後重新載入。 |
| `acceptChanges()` | 清空 `updatedRows`（拖曳異動記錄）。 |
| `getChangedDatas()` | 回傳拖曳造成的異動資料陣列（`updated`）。 |
| `submit()` | 將拖曳異動透過 `$.updateData` 提交至伺服端。 |

### 拖曳行為

- `nodedrop` 事件觸發時更新被拖曳節點的 `parentField` 為目標節點的 `idField`，並記錄至 `updatedRows`。
- 若 `verticalLevel` 有值且涉及垂直層級的拖曳，重新 `loadData` 以正確渲染。
- `onBeforeDrop(dragData, dropData)` 回傳 `false` 可取消拖曳。

## 備註

- OrgChart 是唯一使用 ToolItems（工具列按鈕）的圖表元件，按鈕渲染在圖表上方的 `btn-group` 中。
- `verticalLevel` 控制從第幾層開始改為垂直排列，適合深層組織結構的空間最佳化。
- `editForm` 限定只能選擇 `dataform` 類型的元件，點擊節點可開啟表單編輯。
- `nodeTemplate` 是腳本編輯器，可自訂節點的 HTML 樣板，參數 `data` 包含該節點的資料列。
- 按鈕的 `btnCls` 若為空或 `btn-default`，會自動改用 `datagrid-btn` 樣式。
