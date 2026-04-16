# Creator (Editor)

> `EEPRWDTools.Core/Editors/Creator.cs` — 22 行
> 繼承：`RWDEditor`

## 用途

**建立者欄位編輯器**。自動記錄資料建立者資訊，欄位預設為唯讀。可搭配日期欄位記錄建立時間。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **dateField** | string | 欄位選擇器 `[ColumnEditor("parent")]` | — | 對應的建立日期欄位 |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.creator`（`bootstrap.infolight.js` 第 13434–13507 行）

### 初始化

- 解析欄位名稱（`field`），設為 `disabled`（唯讀）。

### 主要方法

| 方法 | 說明 |
|------|------|
| `getDateValue(row)` | 將當前登入者（`sessionStorage.clientInfo.user`）寫入 `row[field]`，將當前時間（`yyyy/MM/dd hh:mm:ss`）寫入 `row[dateField]`。供新增資料時自動填入。 |
| `getOldFormatterDateValue(row)` | 將 `row[dateField]` 從 ISO 格式轉為 `yyyy/MM/dd hh:mm:ss` 格式（移除 T、Z，替換 `-` 為 `/`）。 |
| `getValue()` | 回傳 `data('value')`。 |
| `setValue(value)` | 設定 `data('value')` 及顯示值。 |
| `setDateValue(row)` | 呼叫 `formatter` 格式化後設為顯示值。 |
| `formatter(row)` | 預設格式：`"使用者 (日期時間)"`。若有自訂 `formatter` 函式則呼叫之。欄位或日期為空時回傳 `row[field]`。 |
| `readonly()` | 設為 `disabled`。 |

## 備註

- Render 時自動加入 `readonly: true` 到 data-options。
- 渲染為 `<input>` 加上 `bootstrap-creator` CSS 類別。
