# EEP SP7 Schedule 排程機制

> EEP 內建**兩套並行**的排程執行機制：獨立 exe（`Schedule.Core`）與 class library（`ScheduleHelper`）。兩者邏輯幾乎相同、讀同一張 DB 表、呼叫同一組方法，差別在於**部署方式**。

## 兩種機制速覽

| 項目 | 方式 A：`Schedule.Core.exe` | 方式 B：`ScheduleHelper` |
|------|---------------------------|--------------------------|
| 類型 | 獨立 Console Application | Class Library（`EEPServerTools.Core`） |
| 原始碼 | `Schedule.Core/Program.cs` | `EEPServerTools.Core/Utility/ScheduleHelper.cs` |
| 啟動方式 | `Schedule.Core.exe` 直接執行（或包成 Windows Service） | 其他 host 程式呼叫 `new ScheduleHelper().StartSchedule()` |
| 部署 | 獨立可執行檔 + `App.config` | 內嵌於呼叫它的主機程式中 |
| 程序獨立性 | ✅ 獨立 process | ❌ 與 host 共享 process |
| 與 Web 相依 | `App.config` 的 `WebSitePath` 指向 EEP web 目錄載入 assembly | 直接在同 process 內使用 |

## 共同的核心邏輯

兩者都執行相同的 6 步：

```
1. ClearLastTime()
   UPDATE SYS_SCHEDULE SET LastTime = NULL
     WHERE LastTime = <DateTime.MaxValue 的 Unix timestamp>
       AND Type = 'interval'
   （清理上次啟動時「執行中」的 long-interval 標記）

2. 30 秒 Timer
   System.Timers.Timer { Interval = 30000 }

3. 每次 tick：Start()
   讀 SYS_SCHEDULE → 對每筆呼叫 IsScheduleActive(now, row)

4. 判斷是否啟動
   - Disabled = true → false
   - Type = 'interval'：
       * 若 InvokeTime 有 start/end 時窗（"HH:mm,HH:mm"），目前時間不在窗內 → false
       * (now 的 Unix ms) - LastTime >= interval * 60 * 1000 → true
   - Type = 'weekly'：
       * 當天 DayOfWeek (0-6) 在 Setting 清單中 + HH:mm 剛好符合 InvokeTime
   - Type = 'monthly'：
       * 當天 Day (1-31) 在 Setting 清單中 + HH:mm 剛好符合 InvokeTime

5. 若 active → Thread.Start(Invoke)（非同步、各 schedule 獨立 thread）

6. Invoke：
   - 解析 Method 欄位：
       * "ModuleName.MethodName" → new DataModule(clientInfo, moduleName).CallMethod(methodName, parameters)
       * "PROC.ProcessorName"    → new DataModule(clientInfo, "PROC").CallProcessorMethod(processorName, parameters)
   - 成功 → LogSchedule(startTime, row, result) 更新 LastTime、寫 SYS_SCHEDULE_LOG
   - 失敗 → LogSchedule(..., null, exception) 記 Error 欄位
```

## 方式 A：`Schedule.Core.exe`（獨立執行）

### 檔案與部署

| 檔案 | 用途 |
|------|------|
| `Schedule.Core/Program.cs` L1-318 | Console app 進入點 + 所有排程邏輯 |
| `Schedule.Core/Schedule.Core.csproj` | `<OutputType>Exe</OutputType>`、依賴 `EEPBase.Core` / `EEPServerTools.Core` |
| `Schedule.Core/App.config` | `<appSettings><add key="WebSitePath" value="..."/></appSettings>` |

### 啟動流程

```
Schedule.Core.exe
  ↓
Main()
  ├─ 讀 App.config 的 WebSitePath
  ├─ System.IO.Directory.SetCurrentDirectory(WebSitePath)
  │     → 這樣才能載入 EEP 的相對路徑 assembly、讀到對應 appsettings.json
  ├─ ClearLastTime()
  └─ timer.Start()（30s 一次，永遠跑）
     while (!Console.ReadLine().ToUpper().Contains("CLOSE")) continue;
```

### 適合的情境

- **獨立伺服器**部署（與 Web 分機器）
- 不想讓 Web 應用背負排程工作（避免影響 Web 請求回應時間）
- 可以用 Windows 工作排程器 / Linux systemd / sc.exe 包成服務
- 程序 crash 不影響 Web；重啟只要重跑 exe

### 部署範例（Windows）

```bat
REM 1. 發布 Schedule.Core 為 self-contained exe
dotnet publish Schedule.Core.csproj -c Release -r win-x64 --self-contained

REM 2. 放到伺服器（例：D:\EEP\Schedule\）
REM 3. 設定 App.config 的 WebSitePath 指向 EEP 網站目錄
REM 4. 註冊成 Windows Service
sc create "EEP Schedule" binPath= "D:\EEP\Schedule\Schedule.Core.exe" start= auto
sc start "EEP Schedule"
```

### 注意事項

- **`WebSitePath` 必填** — 否則載入 assembly 失敗、資料庫連線字串讀不到
- **Console.ReadLine() 會阻塞** — 若用 Windows Service 包，要把這行改掉或用不同 host
- **無內建檔案 log** — Console 輸出需靠 Service 重導向到檔案（`> log.txt 2>&1` 或 systemd 的 StandardOutput）

## 方式 B：`ScheduleHelper`（Class Library）

### 檔案

`EEPServerTools.Core/Utility/ScheduleHelper.cs` L15-303

### 關鍵 API

```csharp
public class ScheduleHelper
{
    public void StartSchedule()
    {
        ClearLastTime();
        System.Timers.Timer timer = new System.Timers.Timer() { Interval = 30000 };
        timer.Elapsed += Timer_Elapsed;
        timer.Start();
    }

    // 其餘 private 方法：Start, IsScheduleActive, Invoke, LogSchedule, EnableLongInterval, ClearLastTime, GetTimestamp
}
```

### 使用方法

**選項 B-1：整合進 EEPWebClient.Core（Web 內嵌）**

在 `Program.cs` / `Startup.cs` 啟動時呼叫：

```csharp
// Program.cs（.NET 6+）
var app = builder.Build();

// ... 其他 middleware 註冊 ...

new ScheduleHelper().StartSchedule();   // ← 排程 thread 與 Web 共存

app.Run();
```

優點：單一部署單位，不用額外跑 exe。
缺點：Web 應用重啟（如 IIS recycle、升版）會中斷排程；排程任務佔用 Web process 資源。

**選項 B-2：包成自訂 Windows Service（無 exe + App.config）**

```csharp
// SomeService/Program.cs
using EEPServerTools.Core.Utility;

class Program
{
    static void Main()
    {
        // 可以設定目錄、載入 appsettings.json、DI 等
        new ScheduleHelper().StartSchedule();

        Console.WriteLine("Press Ctrl+C to stop");
        System.Threading.Thread.Sleep(Timeout.Infinite);
    }
}
```

**選項 B-3：包成 ASP.NET Core `BackgroundService`**

```csharp
public class ScheduleBackgroundService : BackgroundService
{
    protected override Task ExecuteAsync(CancellationToken stoppingToken)
    {
        new ScheduleHelper().StartSchedule();
        return Task.CompletedTask;
    }
}

// Program.cs
builder.Services.AddHostedService<ScheduleBackgroundService>();
```

### 適合的情境

- 規模較小、不想多一個 exe 部署
- 有自訂 host 程式（例如公司內部整合的服務容器）
- 想跟 Web 應用一起升版、一起 rolling restart

### 注意事項

- 與 Web 共用 process → **Web 吃記憶體或 GC 抖動會影響 30s timer 觸發**
- **沒有內建 ClearLastTime 的防重入**：若 host 被短時間重啟兩次，中間可能漏掉排程視窗
- `ScheduleHelper` 實例是 instance 方法，但內部的 `LastTime` 是 **static**（L78）— 同一個 host process 只能有一個 ScheduleHelper 實體在跑，不要重複 `new`

## 兩個檔案的關係

`Schedule.Core/Program.cs` 的邏輯與 `ScheduleHelper.cs` **幾乎是複製貼上**（相同的 `Start / IsScheduleActive / Invoke / LogSchedule / ClearLastTime` 方法，差在 `Program.cs` 用 `static` 方法、`ScheduleHelper` 用 instance 方法）。

**這是雙軌維護的設計** — 修改排程邏輯時兩邊都要改，否則容易分歧。

| 差異點 | `Program.cs` (exe) | `ScheduleHelper.cs` |
|-------|---------------------|---------------------|
| 進入點 | `static Main(string[] args)` | `public void StartSchedule()` |
| 方法修飾 | 全部 `static` | Instance method（但 `LastTime` 仍 static） |
| WebSitePath 載入 | 有（`SetCurrentDirectory`） | 無（由 host 負責） |
| Console Log | 有（同時有 `Log` / `LogError`） | 有（完全相同實作） |
| 外部終止 | `Console.ReadLine()` 等 `CLOSE` | 無（由 host 控制生命週期） |

## SYS_SCHEDULE 表欄位

| 欄位 | 類型 | 說明 |
|------|------|------|
| **ID** | int identity | 主鍵 |
| **Name** | nvarchar(50) | 排程名稱（顯示用） |
| **Type** | varchar(20) | `interval` / `weekly` / `monthly` |
| **Setting** | text | Type 對應設定：<br>- interval：**分鐘數**（int）<br>- weekly：`0,2,4` 星期幾（0=Sun, 6=Sat，逗號分隔）<br>- monthly：`1,15` 幾號（1-31，逗號分隔） |
| **InvokeTime** | text | 執行時間：<br>- interval：`HH:mm,HH:mm` 開窗/閉窗時段（空字串 = 全天可跑）<br>- weekly / monthly：`HH:mm` 執行時間（支援多個，逗號分隔） |
| **Method** | nvarchar(50) | 要執行的方法：<br>- `ModuleName.MethodName` — 呼叫模組 ServerMethod<br>- `PROC.ProcessorName` — 呼叫 Processor |
| **Parameter** | text | JSON 物件字串，轉 `JObject` 傳給方法 |
| **UserID** | varchar(20) | 建立者（顯示用） |
| **Solution** | nvarchar(20) | 方案（多租戶隔離，傳入 `ClientInfo.Solution`） |
| **DBAlias** | nvarchar(50) | 目標資料庫別名（傳入 `ClientInfo.Database`） |
| **LastTime** | bigint | 上次成功執行的 Unix timestamp（毫秒）；NULL = 從未執行 |
| **Disabled** | bit | true 時跳過 |

## SYS_SCHEDULE_LOG 表欄位

| 欄位 | 類型 | 說明 |
|------|------|------|
| **ID** | int | 對應 SYS_SCHEDULE.ID |
| **StartDatetime** | datetime | 執行開始時間（`yyyy/MM/dd HH:mm:ss`） |
| **Duration** | int | 執行毫秒數（endTime - startTime） |
| **Description** | text | 成功時：方法的回傳值（字串或 JSON serialize） |
| **Error** | text | 失敗時：`Exception.Message`（會遞迴取 `InnerException.Message`） |

## Method 執行路徑

```
Invoke(startTime, row)
  ↓
解析 row["Method"]
  ↓
  ├─ "MODULE.METHOD" →
  │    new DataModule(new ClientInfo {
  │        Database = row["DBAlias"],
  │        Solution = row["Solution"]
  │    }, "MODULE")
  │    .CallMethod("METHOD", parameters)
  │
  └─ "PROC.NAME" →
       new DataModule(clientInfo, "PROC", new List<Component>())
       .CallProcessorMethod("NAME", parameters)
  ↓
result = object（字串或任意可 serialize 物件）
  ↓
LogSchedule(startTime, row, result)
  → UPDATE SYS_SCHEDULE SET LastTime = <timestamp>
  → INSERT SYS_SCHEDULE_LOG
```

## Long-Interval 特殊處理（防重入）

對 `Type = interval` 且**間隔 ≤ 60 分鐘**的排程，採取「執行中」標記保護（`EnableLongInterval()` L274-287）：

1. Invoke 開始時，先 `LogSchedule(DateTime.MaxValue, schedule)` — 把 LastTime 暫標成 `DateTime.MaxValue`
2. 若 30 秒後 timer 又觸發這筆 — 因為 `now - MaxValue < 0`，會判定「不該執行」，避免重複啟動
3. Invoke 完成時，`LogSchedule(startTime, schedule, result)` 更新 LastTime 為**結束時間**（不是開始時間，避免下次太快再觸發）

重啟時 `ClearLastTime()` 會清掉所有 `LastTime = DateTime.MaxValue` 的標記（清掉上次 crash 殘留），避免啟動後任何 interval 排程都卡住不跑。

## 設計端介面：`ScheduleProvider`

**另一個相關元件**：`EEPGlobal.Core/Provider/ScheduleProvider.cs`

此元件**不執行排程**，僅提供設計端 CRUD：
- 路由：`POST /design/` + `type=schedule`
- 模式：
  - `load` — 列出當前 Solution + DBAlias 的排程
  - `save` — 新增/修改排程定義
  - `logs` — 讀取某排程的執行紀錄
  - `clearLogs` — 清除執行紀錄

實際執行仍需要 **Schedule.Core.exe** 或 **ScheduleHelper**。

## 兩種方式的選擇建議

| 情境 | 建議 |
|------|------|
| 單一伺服器、省部署 | **方式 B（ScheduleHelper + BackgroundService）** 嵌入 Web |
| 大量/長時間排程任務，不想影響 Web | **方式 A（Schedule.Core.exe）** 獨立部署 |
| 有專用整合服務容器 | **方式 B**，自訂 host 包 ScheduleHelper |
| 跨伺服器負載平衡（Web 集群） | **方式 A 單機啟動**，避免集群每個 node 都跑一份重複執行 |
| 升級時要完全隔離 | **方式 A**，Web 升版不影響排程 |

> 若用方式 B 且 Web 是**多 instance / 集群**，必須只在**一台 node 啟動 ScheduleHelper**（或另外做 leader election），否則每個 node 都會跑 timer → 同一筆排程會執行 N 次。

## 常見陷阱

| 陷阱 | 說明 | 對策 |
|------|------|------|
| **雙軌邏輯分歧** | `Program.cs` 與 `ScheduleHelper.cs` 幾乎複製貼上，改一邊忘另一邊 | 改排程邏輯時兩個檔都要改 |
| **多 instance Web 啟用 ScheduleHelper** | 每個 node 都跑，任務被執行 N 次 | 只在一台 node 啟動，或轉用 Schedule.Core.exe |
| **`LastTime` 是 static** | 同 process 不要 `new ScheduleHelper()` 多次 | 啟動期只 new 一個 |
| **weekly/monthly 時間要剛好符合 HH:mm** | 內部比對的是 `startTime.ToString("HH:mm")`，只要當分鐘該到 | 30s timer + 同分鐘內會檢查兩次，第二次被 static `LastTime` 擋 |
| **interval 時窗寫錯** | `InvokeTime` 格式是 `"08:00,18:00"`，不是 JSON 陣列 | 設定時逗號分隔兩個 HH:mm |
| **`Console.ReadLine()` 阻塞** | 方式 A 用 Windows Service 包時會卡住 | 改用 `while(true) Thread.Sleep(Timeout.Infinite);` 或 `await Host.WaitForShutdown()` |
| **Disabled 不是布林欄位型別** | 比對用 `string.Compare(..., bool.TrueString, true) == 0`，即字串 `"True"` / `"true"` | 資料庫存入時確認是字串 True/False |
| **Schedule.Core 沒有檔案 log** | 只輸出 Console | Windows Service 用時 stdin/stdout 需重導向 |

## 相關討論區

| ID | 主題 |
|----|------|
| [#476301](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=476301) | 排程相關問題（虹光精密） |

## 參考檔案路徑

- 獨立 exe 進入點：`Schedule.Core/Program.cs`
- class library：`EEPServerTools.Core/Utility/ScheduleHelper.cs`
- 設計端管理：`EEPGlobal.Core/Provider/ScheduleProvider.cs`
- 系統表 SQL 定義：相關 `systemTables.sql`
- 相關資料表文件：
  - [SYS_SCHEDULE](../EEP%20Core系統資料表/SYS_SCHEDULE.md)
  - [SYS_SCHEDULE_LOG](../EEP%20Core系統資料表/SYS_SCHEDULE_LOG.md)
