# Htmleditor (Editor)

> `EEPRWDTools.Core/Editors/Htmleditor.cs` — 31 行
> 繼承：`RWDEditor`

## 用途

**HTML 富文字編輯器**。提供所見即所得的 HTML 編輯功能，支援圖片插入。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 200 | 編輯器高度（px） |
| **imageHeight** | int | 數字框 `[NumberboxEditor]` | 150 | 插入圖片預設高度（px） |
| **imageFolder** | string | 文字 | — | 圖片上傳資料夾 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **htmlAvailable** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否允許直接輸入 HTML |

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 15886–15999 行
> jQuery 外掛名稱：`$.fn.htmleditor`

### 初始化流程

1. 解析 `data-options`，以元件 ID 呼叫 `UM.getEditor(id, config)` 初始化 UMeditor（UEditor Mini）。
2. UMeditor 設定：
   - **圖片上傳路徑**：若有 `imageFolder` 則為 `../../file/umimage?f={imageFolder}`，否則為 `../../file/umimage`。圖片讀取路徑統一為 `../../file`。
   - **圖片高度**：使用 `imageHeight` 屬性。
   - **工具列**：若 `htmlAvailable` 為 true，工具列包含 `source`（原始碼編輯）按鈕；否則不含。兩者皆包含：復原、取消復原、粗體、斜體、底線、刪除線、前景色、背景色、清除格式、全選、清空、字型、字級、對齊、連結、圖片。
   - **寬度**：`100%`。
   - **高度**：使用 `height` 屬性。
   - **imageScaleEnabled**：`false`（禁止縮放圖片）。
3. 監聽 `afteruiready` 事件：若在 UMeditor 就緒前已有 `setValue` 或 `readonly` 呼叫，於此時補套用（透過 `$body.data('value')` / `$body.data('readonly')` 暫存機制）。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化 UMeditor |
| `getValue` | 取得 HTML 內容。將 `_src="..."` 屬性轉換為 `onclick="$.showPreviewDialog('...')"` 供預覽使用 |
| `setValue(value)` | 設定 HTML 內容。先 `reset()` 再 `setContent()`。反向將 `onclick="$.showPreviewDialog('...')"` 轉回 `_src="..."`。若 UMeditor 尚未就緒，暫存至 data |
| `readonly(value)` | 設定唯讀狀態。`true` 呼叫 `um.setDisabled()`、`false` 呼叫 `um.setEnabled()`。若 UMeditor 尚未就緒，暫存至 data |

### 圖片插入擴展

在 `$.fn.htmleditor` 外部（第 15984–15999 行），若全域 `UM` 物件存在，會覆寫 `UM.commands.insertimage.execCommand`：插入圖片時自動套用 `imageHeight` 設定的高度至每張圖片。

## ⚠️ 注意事項：ValidateXss 必須前後端都關閉

使用 Htmleditor 時，因為編輯器產生的內容包含 HTML 標籤（`<b>`、`<img>`、`<a>` 等），會被 EEP 的 XSS 驗證攔截導致存檔失敗。**必須同時關閉前後端的 XSS 驗證**：

### 後端（Server 元件）

在模組 JSON 的 UpdateComponent 設定 `validateXss: false`：

```json
{
  "type": "updatecomponent",
  "id": "ucMain",
  "infocommand": "cmdMain",
  "validateXss": false
}
```

### 前端（RWD 元件）

在 DataGrid 或 DataForm 的 Column 設定中，確認含有 Htmleditor 的欄位不會被前端 XSS 檢查攔截。若整個表單都有 HTML 欄位，也可在 DataForm/DataGrid 層級設定。

### 為什麼需要兩端都關

| 層級 | 檢查位置 | 程式碼 |
|------|---------|--------|
| **前端** | DataGrid `endEdit` → `validateRow` | `$.xssValidate(value)` |
| **後端** | UpdateComponent `GetInsertSql` / `GetUpdateSql` | `ValidateValue(v)` — 正則驗證 |

任一端未關閉都會導致含 HTML 標籤的內容被拒絕（前端彈出 XSS 警告、後端拋出 `validateXss` 例外）。

## 備註

- 渲染為 `<div>` 加上 `bootstrap-htmleditor` CSS 類別。
- 使用 UMeditor（UEditor Mini）作為富文字編輯器核心。
- `_src` ↔ `onclick` 屬性互轉是為了讓圖片在檢視模式下可點擊預覽，編輯模式下保留原始路徑。
