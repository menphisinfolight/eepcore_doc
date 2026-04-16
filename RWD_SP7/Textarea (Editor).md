# Textarea (Editor)

> `EEPRWDTools.Core/Editors/Textarea.cs` — 36 行
> 繼承：`RWDEditor`

## 用途

**多行文字輸入框編輯器**。用於需要輸入較長文字的欄位，渲染為 `<textarea>` 標籤。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **maxLength** | int? | 數字框 `[NumberboxEditor]` | — | 最大輸入長度 |
| **rows** | int | 數字框 `[NumberboxEditor]` | 3 | 顯示行數 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 前端行為（JavaScript）

Textarea 沒有獨立的 `$.createObj` 元件，而是透過 **datagrid / tablegrid 的 `editors.textarea`** 定義來初始化（`$.fn.datagrid.defaults.editors.textarea`）。

### 初始化（init）

在 DataGrid 編輯欄位容器中，建立的並非 `<textarea>`，而是一個 **`<input type="text">`**。當使用者 **focus 該輸入框時**，會動態彈出一個 Modal 對話框，內含真正的 `<textarea>`（固定高度 200px）進行多行文字編輯。

**Modal 行為：**
1. focus 時呼叫 `$.createModal()` 建立對話框，包含「確定」和「取消」兩個按鈕。
2. 若有 `maxLength` 選項，會設定 Modal 內 `<textarea>` 的 `maxlength` 屬性。
3. Modal 加上 `refval-modal` CSS 類別。
4. `shown.bs.modal` 事件中將目前值填入 `<textarea>` 並自動 focus。
5. 按「確定」後將 `<textarea>` 的值寫回 `<input>` 的 `val()` 與 `data('value')`。
6. `hidden.bs.modal` 事件中自動移除 Modal DOM。

### getValue / setValue

- `getValue`：回傳 `$(target).data('value')`（非 input 的 val，因實際值存在 data 中）。
- `setValue`：同時設定 `$(target).val(value)` 和 `$(target).data('value', value)`。

### readonly

透過 `$(target).attr('disabled', value)` 切換唯讀狀態。

> **註：** 在 DataGrid 中 Textarea 的 UX 模式是「點擊欄位 → 彈出 Modal 編輯」，而非直接在儲存格內顯示多行文字。

## 備註

- 當 `maxLength` 有值且大於 0 時，會在 HTML 上設定 `maxlength` 屬性。
