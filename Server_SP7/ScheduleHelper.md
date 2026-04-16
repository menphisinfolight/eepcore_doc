# ScheduleHelper

> `EEPServerTools.Core/Utility/ScheduleHelper.cs` — 303 行

## 用途

**排程任務執行器**（Schedule Task Executor）。

ScheduleHelper 負責 EEP Core 的伺服器端定時排程管理。啟動後以 30 秒為週期輪詢 `SYS_SCHEDULE` 資料表，檢查每個排程是否到達執行時間，符合條件者以獨立執行緒呼叫指定的模組方法或 Processor，並將執行結果寫入排程日誌。

## 核心方法

| 方法 | 參數 | 回傳 | 說明 |
|------|------|------|------|
| `StartSchedule()` | — | void | 啟動排程：清除鎖定旗標後建立 30 秒 Timer |
| `Start()` | — | void | Timer 觸發時的主邏輯：查詢所有排程、逐一檢查是否啟動 |
| `IsScheduleActive(startTime, schedule)` | DateTime, DataRow | bool | 判斷排程是否該在此時間點執行 |
| `Invoke(startTime, row)` | DateTime, DataRow | void | 執行排程任務（呼叫 DataModule.CallMethod 或 CallProcessorMethod） |
| `LogSchedule(startTime, schedule, result, e)` | DateTime, DataRow, object, Exception | void | 寫入執行日誌（更新 LastTime + 新增 scheduleLog） |
| `EnableLongInterval(schedule)` | DataRow | bool | 判斷是否啟用長時間間隔鎖定機制 |
| `ClearLastTime()` | — | void | 啟動時清除被鎖定（LastTime = MaxValue）的 interval 排程 |
| `GetTimestamp(dateTime)` | DateTime | long | 將 DateTime 轉為毫秒級 Unix 時間戳（自 1970-01-01 起算） |
| `FormatScheduleName(schedule)` | DataRow | string | 格式化排程名稱，如 `#1 MyJob@DBAlias-Solution` |

## 關鍵邏輯

### 30 秒輪詢機制

```
StartSchedule()
  → ClearLastTime()          // 清除上次異常中斷的鎖定
  → Timer (Interval = 30000ms)
      → Start()
          → 讀取 SYS_SCHEDULE 全部排程
          → 逐一 IsScheduleActive() 判斷
          → 符合者 → new Thread(Invoke)  // 每個排程獨立執行緒
```

### 排程類型判斷（IsScheduleActive）

排程分為四種類型，透過 `Type` 和 `Setting` 欄位控制：

#### interval（固定間隔）

| 欄位 | 說明 |
|------|------|
| `Setting` | 間隔分鐘數（整數） |
| `InvokeTime` | 以逗號分隔的兩個值：`開始時間,結束時間`（HH:mm 格式，可為空） |
| `LastTime` | 上次執行的 Unix 時間戳 |

判斷邏輯：
1. 若 `Setting` 解析失敗或 <= 0 → 不執行。
2. 若目前時間早於開始時間或晚於結束時間 → 不執行。
3. 若 `(現在時間戳 - LastTime) >= interval * 60 * 1000` → 執行。

#### daily（每日）

| 欄位 | 說明 |
|------|------|
| `Setting` | 不使用 |
| `InvokeTime` | 以逗號分隔的執行時間點（HH:mm 格式） |

判斷邏輯：現在的 `HH:mm` 是否在 InvokeTime 清單中。

#### weekly（每週）

| 欄位 | 說明 |
|------|------|
| `Setting` | 以逗號分隔的星期幾（0=日, 1=一, ..., 6=六） |
| `InvokeTime` | 以逗號分隔的執行時間點（HH:mm 格式） |

判斷邏輯：今天星期幾在 Setting 中，且 `HH:mm` 在 InvokeTime 中。

#### monthly（每月）

| 欄位 | 說明 |
|------|------|
| `Setting` | 以逗號分隔的日期（1~31） |
| `InvokeTime` | 以逗號分隔的執行時間點（HH:mm 格式） |

判斷邏輯：今天幾號在 Setting 中，且 `HH:mm` 在 InvokeTime 中。

### daily/weekly/monthly 的防重複執行

對於非 interval 類型，使用 static `LastTime` 欄位記錄最後檢查時間（精確到分鐘）。若同一分鐘內 Timer 再次觸發（30 秒內可能觸發兩次同一分鐘），直接跳過，避免重複執行。

### 長時間間隔鎖定機制（EnableLongInterval）

當 interval 類型的間隔 <= 60 分鐘時啟用此機制：

1. **執行前**：將 `LastTime` 設為 `DateTime.MaxValue` 的時間戳，相當於「鎖定」此排程，防止在任務執行期間 Timer 再次觸發同一排程。
2. **執行後**：將 `LastTime` 更新為實際結束時間（`DateTime.Now`），解除鎖定。
3. **服務啟動時**：`ClearLastTime()` 將所有 `LastTime = MaxValue` 且 `Type = "interval"` 的排程重設為 NULL，清除上次異常中斷遺留的鎖定。

> 間隔 > 60 分鐘時不啟用鎖定，因為執行間隔足夠長，不會在任務尚未完成時再次觸發。

### 任務執行（Invoke）

```csharp
// Method 欄位格式
"PROC.ProcessorID"     → DataModule.CallProcessorMethod(ProcessorID, parameters)
"ModuleName.MethodName" → DataModule.CallMethod(MethodName, parameters)
```

- `Parameter` 欄位為 JSON 字串，解析為 JObject 傳入。
- 每次執行建立新的 `ClientInfo`（含 Database、Solution）和 `DataModule`。
- 執行結果（或例外）寫入 `scheduleLog` 表，包含 StartDatetime、Duration（毫秒）、Description（結果）、Error（錯誤訊息）。

## 備註

- Timer 間隔固定 30 秒，無法透過設定調整；但此間隔確保每分鐘至少檢查一次，對 daily/weekly/monthly 類型的 HH:mm 判斷而言足夠。
- 每個符合條件的排程以 `new Thread(ParameterizedThreadStart)` 獨立執行，排程之間不會互相阻塞。
- `Disabled` 欄位為 `"True"` 時排程不會執行。
- 例外處理會追溯到最內層的 `InnerException` 再記錄錯誤訊息。
- `GetTimestamp` 使用的是本地時間（非 UTC），因為 `DateTime.Now` 為本地時間。
- 日誌同時更新 `schedule` 表（LastTime）和新增 `scheduleLog` 表記錄，兩者在同一個 `UpdateDataset` 呼叫中批次處理。
