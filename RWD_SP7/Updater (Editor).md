# Updater (Editor)

> `EEPRWDTools.Core/Editors/Updater.cs` — 20 行
> 繼承：`RWDEditor`

## 用途

**更新者欄位編輯器**。自動記錄資料最後更新者資訊。可搭配日期欄位記錄更新時間。與 Creator 對應。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **dateField** | string | 欄位選擇器 `[ColumnEditor("parent")]` | — | 對應的更新日期欄位 |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.updater`（`bootstrap.infolight.js` 第 13509–13574 行）

### 初始化

- 解析欄位名稱（`field`），設為 `disabled`（唯讀）。

### 主要方法

| 方法 | 說明 |
|------|------|
| `getDateValue(row)` | 將當前登入者寫入 `row[field]`，當前時間寫入 `row[dateField]`。供更新資料時自動填入。 |
| `getValue()` | 解析顯示值（以空白分隔），回傳第一段（使用者名稱）；若只有一段則回傳空字串。 |
| `setValue(value)` | 直接設定顯示值。 |
| `setDateValue(row)` | 呼叫 `formatter` 格式化後設為顯示值。 |
| `formatter(row)` | 預設格式：`"使用者 (日期時間)"`。若有自訂 `formatter` 則呼叫之。無日期時回傳 `row[field]`；兩者皆無時回傳目前輸入框的值。 |
| `readonly()` | 設為 `disabled`。 |

### 與 Creator 的差異

- `getValue()` 邏輯不同：Updater 從顯示值解析使用者名稱，Creator 從 `data('value')` 取值。
- `formatter` 的 fallback 邏輯不同：Updater 在 `field` 為空時使用 `sessionStorage.clientInfo.user` 作為預設使用者。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-updater` CSS 類別。
- 與 Creator 結構相同，但用途不同（記錄最後更新者而非建立者）。
