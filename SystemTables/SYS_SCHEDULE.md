# SYS_SCHEDULE

## 用途

**排程任務定義表**（Scheduled Task Definition）。

SYS_SCHEDULE 儲存系統的排程任務定義，排程引擎每 30 秒掃描此表，根據 Type / Setting / InvokeTime 判斷任務是否需要執行，並透過 DataModule 呼叫指定的 Method。

### 使用場景

| 場景 | 說明 |
|------|------|
| **排程引擎輪詢** | `ScheduleHelper.Start()` / `Schedule.Core.Program.Start()` 每 30 秒讀取所有排程定義，判斷是否需執行 |
| **排程管理（設計端）** | `ScheduleProvider.Load()` 載入當前方案+資料庫的排程定義，`Save()` 儲存修改 |
| **排程日誌查看** | `ScheduleProvider.LoadLogs()` 以 ID 關聯 SYS_SCHEDULE_LOG 查詢執行記錄 |
| **LastTime 更新** | 執行後更新 LastTime 為當前 timestamp，用於 interval 類型的間隔計算 |
| **啟動時清除** | `ClearLastTime()` 在排程引擎啟動時，將 interval 類型且 LastTime = MaxValue 的記錄清除（防止鎖定） |

### 關聯表

```
SYS_SCHEDULE ──(ID)──> SYS_SCHEDULE_LOG  （執行日誌，一對多）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Utility/ScheduleHelper.cs` | 排程引擎（嵌入 Web 應用）：StartSchedule()、IsScheduleActive()、Invoke()、LogSchedule() |
| `Schedule.Core/Program.cs` | 獨立排程執行程式（Console App），邏輯與 ScheduleHelper 幾乎相同 |
| `EEPGlobal.Core/Provider/ScheduleProvider.cs` | 設計端 API：Load()、LoadLogs()、ClearLogs()、Save() |
| `EEPWebClient.Core/design/server/SystemTable.json` | `schedule` infocommand：`SELECT * FROM SYS_SCHEDULE`，keys: `ID` |
| `EEPWebClient.Core/Views/Design/Schedule.cshtml` | 前端排程管理介面 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int IDENTITY(1,1)` PK | NOT NULL | 排程任務識別碼（自動遞增） |
| **Name** | `nvarchar(50)` | NULL | 任務名稱 |
| **Type** | `varchar(20)` | NULL | 排程類型（見下方列舉） |
| **Setting** | `nvarchar(max)` | NULL | 排程設定值（依 Type 而異，見下方說明） |
| **Method** | `nvarchar(50)` | NULL | 執行方法名稱（格式：`模組.方法` 或 `PROC.方法`） |
| **Parameter** | `nvarchar(max)` | NULL | 執行參數（JSON 格式） |
| **InvokeTime** | `nvarchar(max)` | NULL | 執行時間設定（依 Type 而異，見下方說明） |
| **UserID** | `varchar(20)` | NULL | 建立者帳號 |
| **Solution** | `nvarchar(20)` | NULL | 所屬方案 |
| **DBAlias** | `nvarchar(50)` | NULL | 資料庫別名 |
| **LastTime** | `bigint` | NULL | 最後執行時間（Unix timestamp 毫秒） |
| **Disabled** | `bit` | NULL | 是否停用（後期擴充欄位） |

### Type 排程類型與 Setting / InvokeTime 的關係

| Type | Setting 意義 | InvokeTime 意義 |
|------|-------------|----------------|
| `interval` | 間隔分鐘數（如 `"30"` = 每 30 分鐘） | `開始時間,結束時間`（如 `"08:00,18:00"`，空白表示不限） |
| `daily` | 不使用 | `HH:mm[,HH:mm,...]`（每日執行時間，可多個） |
| `weekly` | 星期幾（`0`=日,`1`=一,...`6`=六，逗號分隔） | `HH:mm[,HH:mm,...]` |
| `monthly` | 每月幾日（逗號分隔，如 `"1,15"`） | `HH:mm[,HH:mm,...]` |

### Method 執行方式

| 格式 | 說明 |
|------|------|
| `模組名.方法名` | 透過 `DataModule.CallMethod()` 呼叫指定模組的方法 |
| `PROC.方法名` | 透過 `DataModule.CallProcessorMethod()` 呼叫 Processor |

### 跨資料庫差異

#### 欄位存在度

五個 DB `CREATE TABLE` 都含完整 12 欄位（含 SP7 新增的 `Disabled`）。

#### 型別對照

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `ID` | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` | `SERIAL` |
| `Setting` | `nvarchar(max)` | `clob` | `text` | `NVARCHAR(7000)` | `LVARCHAR(7000)` |
| `Parameter` | `nvarchar(max)` | `clob` | `text` | `NVARCHAR(7000)` | `LVARCHAR(7000)` |
| `InvokeTime` | `nvarchar(max)` | `clob` | `text` | `NVARCHAR(2000)` ⚠️ | `LVARCHAR(2000)` ⚠️ |
| `LastTime` | `bigint` | `number` | `bigint` | `BIGINT` | `BIGINT` |
| `Disabled` | `bit` | `CHAR(1)` | `bit` | `DECIMAL(1,0)` | `DECIMAL(1,0)` |

> ⚠️ DB2 / Informix `InvokeTime` 只有 2000 字元（其他 DB 可無限）。若一個排程的執行時刻字串很長（如每日多個時段 × weekly/monthly 的複雜組合）會被截斷。

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `Disabled` | ✅ | ❌ | ❌ | ❌ | ❌ |

只有 MSSQL 有 ALTER；其他 4 DB CREATE TABLE 已含 `Disabled`，舊版升級時需手動補（含正確的型別對應）。

> ℹ️ 詳細機制與 Oracle 欄位名大小寫陷阱，請見「其他主題 → 排程機制」說明文件（**Schedule排程機制**）。

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 排程判斷邏輯（IsScheduleActive）

```
1. 若 Disabled = true → 不執行
2. 若 Type = "interval"：
   a. 解析 Setting 為間隔分鐘數
   b. 檢查當前時間是否在 InvokeTime 的開始~結束範圍內
   c. 計算 (當前timestamp - LastTime) >= 間隔 × 60 × 1000
3. 若 Type = "daily" / "weekly" / "monthly"：
   a. 防重複：同一分鐘內不重複觸發（LastTime 記錄）
   b. weekly：檢查 Setting 中是否包含今天的星期值
   c. monthly：檢查 Setting 中是否包含今天的日期
   d. 檢查 InvokeTime 中是否包含當前 HH:mm
```

## Long Interval 防鎖機制

interval 類型且間隔 ≤ 60 分鐘時（`EnableLongInterval`），執行前先將 LastTime 設為 `DateTime.MaxValue` 的 timestamp，防止同一任務被重複觸發。執行完成後再更新為實際結束時間。排程引擎啟動時 `ClearLastTime()` 會清除這些鎖定記錄。

---

## 資料生命週期

```
建立：Schedule.cshtml → ScheduleProvider.Save() → INSERT（自動填入 DBAlias + Solution）
載入：ScheduleProvider.Load() → WHERE DBAlias = @DB AND Solution = @Solution
執行：ScheduleHelper.Start() → 每 30 秒掃描 → IsScheduleActive() → Invoke()
      → DataModule.CallMethod() 或 CallProcessorMethod()
      → LogSchedule() → UPDATE LastTime + INSERT SYS_SCHEDULE_LOG
停用：設定 Disabled = true，排程引擎跳過
```

---

## 備註

- 排程引擎有兩個實作：`ScheduleHelper`（嵌入 Web 應用）和 `Schedule.Core.Program`（獨立 Console App），邏輯幾乎相同，差異在防重複機制（ScheduleHelper 用靜態 LastTime，Program 用 Dictionary）。
- LastTime 使用 Unix timestamp 毫秒（自 1970-01-01 起），非標準 DateTimeOffset。
- Save() 時會自動填入當前使用者的 DBAlias 和 Solution，不需前端傳入。
- Disabled 為後期擴充欄位，SQL 腳本中有 ALTER TABLE 補欄位邏輯。
