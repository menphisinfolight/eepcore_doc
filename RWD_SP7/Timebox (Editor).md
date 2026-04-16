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

## 備註

- `dataType` 為 `varchar6` 時，以 6 碼字串儲存（如 `143000` 表示 14:30:00）。
