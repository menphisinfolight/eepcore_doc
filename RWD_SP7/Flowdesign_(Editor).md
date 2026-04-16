# Flowdesign (Editor)

> `EEPRWDTools.Core/Editors/Flowdesign.cs` — 18 行
> 繼承：`RWDEditor`

## 用途

**流程設計編輯器**。用於 EEP 簽核流程相關的欄位，無額外屬性。

## 設計介面屬性

無自訂屬性（僅繼承 RWDEditor 的 Id、Name）。

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.flowdesign`（`bootstrap.infolight.js` 第 19955–20462 行）

### 初始化流程

1. 將 `<input>` 包裹在 `input-group` 結構中，設為 `readonly`。
2. 在輸入框後方加入齒輪按鈕（`glyphicon-cog`），點擊開啟流程設計 Modal。

### 主要方法

| 方法 | 說明 |
|------|------|
| `openModal()` | 開啟流程設計對話框。上方為可拖曳的活動類型工具列（StandActivity、MultiApproveActivity），下方為流程區域（StartActivity → ... → EndActivity）。 |
| `openEditModal(item)` | 編輯單一活動屬性：標題（text）、簽核對象類型（role/user）、指定角色或使用者。MultiApproveActivity 使用 `selectoptions`（多選清單），一般活動使用 `refval`（模糊查詢）。 |
| `getValue()` | 回傳 `$(jq).data('value')`（JSON 字串）。 |
| `setValue(value)` | 儲存值至 `data('value')`，輸入框顯示「(流程資料)」。 |

### 流程設計 Modal 互動

- **拖曳新增**：從工具列拖曳活動類型到流程區域，在目標活動前方插入新活動。
- **上移／下移**：透過 `flow-up`、`flow-down` 按鈕調整活動順序（不可移至 Start 前或 End 後）。
- **刪除**：點擊活動上的 `x` 按鈕，經確認後移除。
- **編輯**：點擊活動開啟 `openEditModal`，可設定活動名稱及簽核者。

### 活動類型

| 類型 | 背景色 | 說明 |
|------|--------|------|
| `StartActivity` | `#e0ff00` | 起始活動（固定） |
| `EndActivity` | `#ff5e00` | 結束活動（固定） |
| `StandActivity` | `#a7ffff` | 一般簽核活動 |
| `MultiApproveActivity` | `#00ffd2` | 多人簽核活動 |

### 資料格式

儲存值為 JSON 陣列，每個元素含：`type`（活動類型）、`text`（標題）、`role`（角色 ID，逗號分隔）、`user`（使用者 ID，逗號分隔）。

### 資料載入

- 開啟 Modal 時透過 `$.Deferred` 並行載入角色（`SystemTable.role`）與使用者（`SystemTable.user`）清單，用於編輯活動時的選取元件及顯示名稱。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-flowdesign` CSS 類別。
- 此為最精簡的 Editor，僅有 Render 實作。
