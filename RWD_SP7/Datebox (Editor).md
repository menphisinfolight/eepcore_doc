# Datebox (Editor)

> `EEPRWDTools.Core/Editors/Datebox.cs` — 39 行
> 繼承：`RWDEditor`

## 用途

**日期選取編輯器**。提供日期選取器（Date Picker），支援多種日期格式和選取器位置設定。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 下拉選項 `[ItemsEditor]` | "yyyy-mm-dd" | 日期格式 |
| **dataType** | DateboxDataType | 列舉 | — | 資料類型 |
| **selectOnly** | bool | 核取方塊 | false | 是否僅能從日曆選取 |
| **pickerPosition** | string | 下拉選項 `[ItemsEditor]` | — | 選取器位置（top-left / top-right / bottom-left / bottom-right） |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(date)]` | — | 選取事件 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 9725–9905 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 取得日期值；`dataType` 為 `varchar8` 時自動轉為 8 碼字串（如 `20250101`） |
| `setValue(value)` | 設定日期值；支援 ISO 格式、`varchar8` 格式，自動依 `format` 格式化顯示 |
| `readonly(bool)` | 同時 disable 輸入框與日曆按鈕 |
| `dateToChar8(value)` | 日期字串轉 8 碼（移除 `-`、`/`） |
| `char8ToDate(value)` | 8 碼字串轉 `yyyy-mm-dd` 格式 |
| `getDateValue(value)` | 依 `format` 將顯示值正規化為 `yyyy/mm/dd` 順序 |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **datetimepicker 整合**：初始化時包裝為 `input-group`，附加日曆圖示按鈕，並建立 `datetimepicker` 實例。
- **自適應彈出位置**：當日曆面板超出視窗邊界時，自動切換為置中 fixed 定位並顯示半透明遮罩（`datebox-backdrop`）。
- **format 處理**：格式字串中 `MM` 統一轉為小寫 `mm`。秒數部分以 `00` 替代。
- **selectOnly 模式**：設定後輸入框為 `readOnly`，僅能透過日曆選取。
- **事件**：`onSelect` 回呼在 `change` 事件時觸發。

## 備註

- 支援格式：`yyyy/mm/dd`、`yyyy-mm-dd`、`yyyy.mm.dd`、`dd/mm/yyyy`、`dd-mm-yyyy`、`mm/dd/yyyy`、`mm-dd/yyyy`。
