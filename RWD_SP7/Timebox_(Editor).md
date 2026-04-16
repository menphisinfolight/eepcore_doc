# Timebox (Editor)

> `EEPRWDTools.Core/Editors/Timebox.cs` — 34 行
> 繼承：`RWDEditor`

## 用途

**時間選取編輯器**。提供時間選取功能，支援設定分鐘間隔和時段範圍。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **minuteStep** | int | 數字框 `[NumberboxEditor]` | 15 | 分鐘間隔 |
| **dataType** | string | 下拉選項 `[ItemsEditor]` | "datetime" | 資料類型（datetime / varchar6） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **minHour** | string | 數字框 `[NumberboxEditor]`（0–23） | — | 最小小時 |
| **maxHour** | string | 數字框 `[NumberboxEditor]`（0–23） | — | 最大小時 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 9907–10003 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 取得時間值；`dataType` 為 `varchar6` 時自動轉為 6 碼字串（如 `143000`） |
| `setValue(value)` | 設定時間值，呼叫 `timepicker('setTime', value)` |
| `readonly(bool)` | 同時 disable 輸入框與時鐘按鈕 |
| `timeToChar6(value)` | 時間字串轉 6 碼（移除 `:`，4 碼補 `00`） |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **timepicker 整合**：初始化時包裝為 `input-group`，附加時鐘圖示按鈕，使用 24 小時制（`showMeridian: false`）。
- **小時範圍限制**：`changeTime` 事件中依 `minHour`、`maxHour` 校正小時值，超出範圍自動修正。
- **防遞迴調整**：透過 `timebox-adjusting` data flag 避免 `setTime` 觸發的 `changeTime` 事件造成無限迴圈。

## 備註

- `dataType` 為 `varchar6` 時，以 6 碼字串儲存（如 `143000` 表示 14:30:00）。
