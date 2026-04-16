# SYS_SCHEDULE_LOG

## 用途

**排程任務執行日誌表**（Schedule Execution Log）。

SYS_SCHEDULE_LOG 記錄排程任務每次執行的結果，包含開始時間、執行結果描述、錯誤訊息、耗時。與 SYS_SCHEDULE 為多對一關係（一個排程任務有多筆執行記錄）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **執行記錄寫入** | `ScheduleHelper.LogSchedule()` / `Program.LogSchedule()` 每次排程執行後 INSERT 一筆記錄 |
| **日誌查詢** | `ScheduleProvider.LoadLogs(id)` 以 ID 關聯查詢指定排程的歷史記錄（分頁，按 StartDatetime DESC） |
| **日誌清除** | `ScheduleProvider.ClearLogs(id)` 刪除指定排程的所有歷史記錄 |

### 關聯表

```
SYS_SCHEDULE_LOG ──(ID)──> SYS_SCHEDULE  （排程定義）
```

透過 `infodatasource idsScheduleLog` 定義主從關係：masterCommand = schedule, detailCommand = scheduleLog, 關聯欄位 = ID。

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Utility/ScheduleHelper.cs` | `LogSchedule()`：執行完成後 INSERT 記錄（ID、StartDatetime、Duration、Description、Error） |
| `Schedule.Core/Program.cs` | `LogSchedule()`：同上，獨立 Console App 版本 |
| `EEPGlobal.Core/Provider/ScheduleProvider.cs` | `LoadLogs()`：分頁查詢日誌；`ClearLogs()`：DELETE WHERE ID = @id |
| `EEPWebClient.Core/design/server/SystemTable.json` | `scheduleLog` infocommand：`SELECT * FROM SYS_SCHEDULE_LOG ORDER BY StartDatetime DESC` |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int` | NOT NULL | 排程任務 ID（對應 SYS_SCHEDULE.ID，非 PK） |
| **StartDatetime** | `datetime` | NULL | 執行開始時間（格式：yyyy/MM/dd HH:mm:ss） |
| **Description** | `nvarchar(max)` | NULL | 執行結果描述（方法回傳值的 JSON 序列化） |
| **Error** | `nvarchar(max)` | NULL | 錯誤訊息（成功時為空字串） |
| **Duration** | `bigint` | NULL | 執行耗時（毫秒，結束 timestamp - 開始 timestamp） |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 |
|------|-----------|-------|--------|-----|
| Description | `nvarchar(max)` | `text` | `clob` | `CLOB (100M)` |
| Error | `nvarchar(max)` | `text` | `clob` | `CLOB (100M)` |
| Duration | `bigint` | `bigint` | `number(20)` | `BIGINT` |
| StartDatetime | `datetime` | `datetime` | `date` | `TIMESTAMP` |

---

## 無主鍵

此表無主鍵定義。同一 ID 會有多筆記錄（每次執行一筆）。

---

## 資料生命週期

```
寫入：排程執行完成
      → LogSchedule(startTime, schedule, result, exception)
      → INSERT INTO SYS_SCHEDULE_LOG (ID, StartDatetime, Duration, Description, Error)
      → 同時 UPDATE SYS_SCHEDULE SET LastTime = @timestamp

查詢：Schedule.cshtml → ScheduleProvider.LoadLogs(id)
      → DataModule.GetDataset("scheduleLog", parentTable=schedule, parentRow={ID})
      → SELECT * FROM SYS_SCHEDULE_LOG WHERE ID = @id ORDER BY StartDatetime DESC

清除：Schedule.cshtml → ScheduleProvider.ClearLogs(id)
      → DatabaseHelper.Delete("SYS_SCHEDULE_LOG", {ID: @id})
```

---

## 備註

- Duration 使用毫秒為單位（Unix timestamp 相減），與 FLOWHISTORY 的 Duration（天數）不同。
- Description 儲存方法回傳值：若回傳 string 直接存入，否則 JSON 序列化。
- Error 在成功時為空字串（非 NULL）。
- 日誌寫入與 LastTime 更新在同一次 DataModule.UpdateDataset 呼叫中完成（同一 transaction）。
