# EEP SP7 Schedule 排程機制

> EEP 內建**兩套並行**的排程執行機制：獨立 exe（`Schedule.Core`）與 class library（`ScheduleHelper`）。兩者邏輯幾乎相同、讀同一張 DB 表、呼叫同一組方法，差別在於**部署方式**。

## ⚠️ 最重要的前提：排程不會自動啟動

**公版 EEP 安裝後，預設不會跑任何排程**。

### 官方 R&D 的建議作法

在 **`EEPWebClient.Core/Startup.cs`** 中以**註解**形式預埋好啟動碼：

```csharp
//var helper = new EEPServerTools.Core.Utility.ScheduleHelper();
//helper.StartSchedule();
```

要啟用 Web 內嵌排程，**取消註解**即可（不用重寫、不用外掛元件）。

### 情境對照

| 情境 | 排程會跑嗎？ |
|------|-------------|
| 只裝 EEP（Web 跑起來）、Startup.cs 維持註解 | ❌ **不會** |
| 部署並執行 `Schedule.Core.exe` | ✅ 會 |
| **取消 Startup.cs 的註解**（Web 內嵌方式） | ✅ 會 |

**結論**：要用排程，**必須選一條路**。兩條都沒做 → SYS_SCHEDULE 裡的排程設定**完全不會執行**，也不會有任何錯誤訊息（因為根本沒有 process 在檢查）。

這是容易誤解的地方 — 看到 SYS_SCHEDULE 表有資料就以為排程有在跑，但其實需要「**有人啟動機制**」才會有排程引擎。

### 版本差異注意

不同 SP7 小版本 Startup.cs 內容可能略有差異：
- **有些版本**已預埋上述兩行註解
- **有些版本**完全沒有（例如 `EEPCore_SP7_TEST` 這個測試版搜尋結果為 No matches）

若拿到的版本找不到預埋的註解，手動加在 `Configure` 方法最末即可（見下方「方式 B」章節）。

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

### ⚠️ 公版 Web 不會自動啟動（需取消註解）

這個 class 提供了 `StartSchedule()` API，**但預設是註解狀態**。

### R&D 官方建議作法：取消 Startup.cs 的註解

有些 SP7 版本的 `EEPWebClient.Core/Startup.cs` 已預埋：

```csharp
//var helper = new EEPServerTools.Core.Utility.ScheduleHelper();
//helper.StartSchedule();
```

要啟用，直接**取消註解**（去掉 `//`）即可：

```csharp
var helper = new EEPServerTools.Core.Utility.ScheduleHelper();
helper.StartSchedule();
```

推薦位置：`Configure(IApplicationBuilder app, IWebHostEnvironment env)` 方法內、`app.UseEndpoints(...)` 之後。

### 若找不到預埋的註解（手動加）

某些 SP7 版本沒有預埋（例：`EEPCore_SP7_TEST`）。此時自己加在 `Configure` 方法最末：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... 既有 middleware 設定 ...

    app.UseEndpoints(endpoints => {
        // ... 既有 routes ...
    });

    // 啟動排程
    new EEPServerTools.Core.Utility.ScheduleHelper().StartSchedule();
}
```

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

## 🔴 Oracle 部署陷阱：欄位大小寫敏感

> 來源：[#479274](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479274) / [#477328](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477328)

### 症狀

- 設計畫面點選「排程」→ 跳 `ORA-00904: "Solution": 無效的識別碼`
- 或執行時報 `ORA-00904: "StartDatetime": 無效的識別碼`
- **MSSQL 環境完全沒事**，只有 Oracle 才炸

### 根本原因

EEP 的 SQL 查詢用了**雙引號識別碼**：

```sql
SELECT * FROM SYS_SCHEDULE WHERE "DBAlias" = N'XXX' AND "Solution" = N'YYY'
```

Oracle 的規則：
- **無引號** → Oracle 自動轉大寫（`DBAlias` 會變 `DBALIAS`）
- **有雙引號** → **大小寫必須完全一致**（`"DBAlias"` ≠ `"DBALIAS"`）

DBA 手動建表時若用了不帶引號的 SQL（`CREATE TABLE SYS_SCHEDULE (DBAlias varchar2(50)...)`），Oracle 自動把欄位名存成 `DBALIAS`，EEP 的雙引號查詢就找不到。

### 正確的建表方式（以引號保住大小寫）

**SYS_SCHEDULE_LOG**（[#479274](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479274) Roland 提供）：

```sql
BEGIN
   createTables('SYS_SCHEDULE_LOG', 'CREATE TABLE SYS_SCHEDULE_LOG(
       ID integer NOT NULL,
       "StartDatetime" date NULL,
       "Description" clob NULL,
       "Error" clob NULL,
       "Duration" integer NULL
   )');
END;
```

**SYS_SCHEDULE**（[#477328](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477328) Roland 提供）：

```sql
BEGIN
   createTables('SYS_SCHEDULE', 'CREATE TABLE SYS_SCHEDULE(
       ID integer NOT NULL,
       "Name" varchar2(50) NULL,
       "Type" varchar2(20) NULL,
       "Setting" clob NULL,
       "Method" varchar2(50) NULL,
       "Parameter" clob NULL,
       "InvokeTime" clob NULL,
       "UserID" varchar2(20) NULL,
       "Solution" varchar2(20) NULL,
       "DBAlias" varchar2(50) NULL,
       "LastTime" varchar2(20) NULL,
       "Disabled" char(1) NULL
   )');
END;
```

### 系統表 SQL 來源檔

不要自己手寫，直接抄這個檔：

```
EEPWebClient.Core/wwwroot/sql/oracle/systemTables.sql
```

裡面有所有系統表的 Oracle 完整 SQL。

### 建立用的 helper procedure

EEP Oracle 部署需要先建好兩個 stored procedure：
- `createTables(tableName, createSQL)` — 一般建表
- `createTablesIdentity(...)` / `createTablesIdentity2(...)` — identity 欄位（自動處理 sequence + trigger）

第一次部署 Oracle 的 EEP 系統時，要先建這幾個 helper proc 才能用上面那些 createTables 呼叫。

### ⚠️ 必須在「System 類型」資料庫執行

> Roland 原話（[#477328](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477328) #4）：「排程功能，只讀取資料庫中，設定為 System 的資料表」

[#479274](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479274) 的客戶踩到的另一個坑：DBA 把建表語法跑在錯的 DB（業務 DB 而非 System DB）→ EEP 還是找不到。

**檢查方式**：到 `/design` → 資料庫設定 → 確認哪個 DB 的 Type 是 `System`，所有 SYS_* 系列建表都要在那個 DB 執行。

### 對策 checklist

部署 Oracle 環境的 EEP 時：

- [ ] 確認 EEP 識別出的 System 資料庫是哪個
- [ ] 拿 `EEPWebClient.Core/wwwroot/sql/oracle/systemTables.sql` 為準
- [ ] 用 `createTables(...)` 包裝建表（SQL 內欄位名要保留雙引號）
- [ ] **不要手寫 `CREATE TABLE` 然後欄位不加引號** — Oracle 會吞引號自動轉大寫
- [ ] 建完 SYS_SCHEDULE / SYS_SCHEDULE_LOG，到 `/design` 點「排程」確認沒跳 ORA-00904

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

## 開發/偵錯體驗差異（實務重點）

這是選擇兩種方式時**最常被忽略但最影響工程效率**的一點。

### 方式 A（`Schedule.Core.exe`）的偵錯痛點

| 問題 | 細節 |
|------|------|
| **斷點難接** | 獨立 process，VS/VSCode 要 `Attach to Process` 選 `Schedule.Core.exe`，且要確保 debug symbol 已載入。包 Windows Service 後更麻煩（Service 起來的 process 非互動登入） |
| **熱重載無效** | 改 ServerMethod 程式碼要重 build Schedule.Core + 重啟 exe，才會載入新 assembly（因為 `WebSitePath` 指向的是 publish 後的目錄） |
| **Console log 可視性差** | 只有 stdout/stderr；包 Service 後要額外重導向到檔案 |
| **例外堆疊追蹤難** | 例外被 LogError 吃掉只留 Message，沒有完整 StackTrace；要改程式加 log 或 attach debugger |
| **與 Web 不同 process 的快取差異** | Web 改過的資料、Session、記憶體快取都看不到 |

實務上客戶回報「排程跑出來結果不對」時，用方式 A 要花很多時間才能定位問題（通常最後都還是改成方式 B 才 debug 完）。

### 方式 B（`ScheduleHelper` 嵌 Web）的偵錯優勢

| 優點 | 細節 |
|------|------|
| **與 Web 同 process 同 debugger** | F5 起 Web 就同時帶起排程，斷點直接可用 |
| **Server Method 可單步執行** | 從 `Invoke` → `CallMethod` → 進入 Module 的 C# 方法，VS 一路 Step In |
| **熱重載更有效** | `dotnet watch run` 或 Hot Reload 改完即時重載（排程也一起更新） |
| **log 與 Web 共用 appsettings 設定** | 可輕鬆整合進 Serilog / Application Insights |
| **快取 / Session 可共用** | 若 ServerMethod 依賴快取或靜態變數，Web 剛寫入的資料排程可讀到 |

### 建議

- **開發 / 測試階段**：**強烈建議用方式 B**（取消 Startup.cs 註解），偵錯時間少一半以上
- **正式環境**：視需求選
  - 小型部署：方式 B 夠用（Web 吃資源通常不算重）
  - 大型 / 跨機器 / 穩定性要求高：方式 A 隔離部署（但排程邏輯或 ServerMethod 若要 debug，**先在開發環境用方式 B 驗證完**再進正式）

> 實務流程：開發用方式 B → 功能驗證完 → 正式環境若採方式 A，只是換部署方式，排程邏輯本身已經 debug 過了。

## 30 秒輪巡的副作用（**官方已知規格**）

> 來源：[#478942](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=478942) / [#481531](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481531) / [#479186](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479186)

### 症狀：同一分鐘排程執行 2 次

多位客戶回報同樣問題：

- 設定每週一 9:30 執行 → **9:30:00 和 9:30:30 各觸發一次**（[#481531](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481531)）
- 每小時執行一次 → 實際**發 2-3 次信**（[#479186](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479186)）
- [#478942](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=478942) 直接點出「同一分鐘執行相同排程二次，有時造成系統錯誤」

### 原因（從原始碼驗證）

Timer 30 秒輪巡一次 → 同一分鐘（HH:mm）內 timer 會觸發 **2 次**（例如 9:30:00 和 9:30:30）。

`ScheduleHelper.IsScheduleActive()` L124-128 對 weekly/monthly 模式用 `static DateTime LastTime` 去重，但 **`LastTime` 存的只到「yyyyMMddHHmm」精度**：

```csharp
if (LastTime.ToString("yyyyMMddHHmm") == startTime.ToString("yyyyMMddHHmm"))
{
    return false;
}
LastTime = startTime;
```

理論上應該只會跑一次，但實務上若**第一次觸發的 Invoke 還沒把 `LastTime` 更新完**，第二次 timer 進來 `IsScheduleActive()` 可能還是看到舊值 → 重複執行。

### 官方回覆（[#478942](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=478942)）

> 「目前 Schedule 中的輪巡方式是 30 秒，因此設定某一特定時間時，是有機率執行到兩次」

**這是規格，非 bug**。

### 對策（客戶端自保）

1. **排程呼叫的 ServerMethod 要設計成 idempotent（冪等）**
   - 寄信前先查「今天是否已發過」
   - 過帳前先查「目標表是否已有本次 key 的記錄」
   - 寫檔前先檢查 file lock
2. **自行加鎖**（自訂 SQL 鎖定記錄 + 檢查）
3. **接受「允許多次執行，結果相同」的設計**
4. **間隔模式（interval）影響較小**（interval 檢查有 `LastTime` 毫秒級時間戳，不會在 30 秒內重複）

## interval 模式以「完成時間」起算

> 來源：[#481152](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481152) / [#476305](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=476305)

### 容易誤解

使用者希望「每 30 分鐘整點執行」（09:00 / 09:30 / 10:00 / 10:30 ...），設定 `Type=interval, Setting=30` 以為會整點觸發。

### 實際行為

`ScheduleHelper.LogSchedule()` L230 更新 LastTime 時：

```csharp
{ "LastTime", EnableLongInterval(schedule)? GetTimestamp(endTime): GetTimestamp(startTime)}
```

- 對**間隔 ≤ 60 分鐘**（`EnableLongInterval` 為 true），`LastTime` 記錄的是**結束時間**（endTime）
- 下次判斷 `now - LastTime >= interval*60*1000` → 間隔**從排程完成那刻起算**

### 例子

- 09:00 排程觸發，執行 5 分鐘
- 09:05 結束 → `LastTime = 09:05`
- 下次 09:05 + 30 分 = **09:35** 才會觸發（不是 09:30）

### 官方態度

> 「會以『完成的時間』起算，沒有進行調整的規劃」

### 對策

要整點執行，**不要用 interval**：
- 改用 `weekly` + 多個 InvokeTime：`"09:00,09:30,10:00,10:30"`（逗號分隔多個 HH:mm）
- 或建多筆排程各設一個時間

## 每月 31 日在 2 月的邊界

> 來源：[#480387](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480387)

### 問題

`Type=monthly, Setting="31"` 希望「每月最後一天執行」。

### 實際行為（原始碼）

`ScheduleHelper.IsScheduleActive()` L131:

```csharp
if (scheduleType == "monthly" && days.IndexOf(startTime.Day.ToString()) < 0) {
    return false;
}
```

**精確比對「今日 Day == 31」**：
- 1 月 / 3 月 / 5 月... → 31 日會跑
- 2 月 / 4 月 / 6 月... → **那個月根本不會有「31 日」→ 永遠不跑**

### 結論

**「31 日」≠「最後一天」**。

### 對策：真的要「每月最後一天」

- 設多個：`Setting="28,29,30,31"` + 在 ServerMethod 內判斷「今天是否為當月最後一天」（`DateTime.DaysInMonth(now.Year, now.Month) == now.Day`）才實際執行
- 或改用 `interval` 每天檢查、ServerMethod 內判斷最後一天才動作

## 方式 A 的 DLL 隔離（客製 dll 要放 Schedule 目錄）

> 來源：[#481544](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481544) / [#480675](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480675) / [#480729](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480729) / [#482128](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482128)

### 為什麼「測試」按鈕能跑、排程自動跑就失敗

- **「測試」按鈕** 是 Web design 頁面發 HTTP 呼叫，實際執行在 **Web process** 裡 → 所有 Web 的 dll 都看得到
- **排程自動執行** 在 `Schedule.Core.exe` 的 **獨立 process** → 只能看到自己 `bin` 目錄下的 dll

若客戶 Server 模組用了自訂 dll（如 `Psod.dll`、`fs` 模組）：
- Web 有 → 測試按鈕 OK ✅
- Schedule.Core 沒 → 自動排程 ❌

### 解法

放 dll 到 **`Schedule.Core\bin\Debug\net8.0`**（或對應 Release 目錄，依 publish 設定）：

```
Schedule.Core/
├─ bin/
│  └─ Debug/net8.0/
│     ├─ Schedule.Core.exe
│     ├─ Schedule.Core.dll
│     ├─ EEPBase.Core.dll
│     ├─ EEPServerTools.Core.dll
│     ├─ {客戶自訂}.dll   ← 手動放這裡
│     └─ ...
```

[#481544](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481544) 官方回覆：「請置放在 `Schedule.Core\bin\Debug\net8.0` 目錄下」

### 這一點也是為什麼「開發階段建議用方式 B」的主要原因

用方式 B（Web 內嵌）→ 客製 dll **自動可見**（反正都在同 process），不用每次更新 dll 還要手動同步到兩個地方。

## SP6 → SP7 升級的 Assembly 問題

> 來源：[#481157](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481157)

### 症狀

升 SP6 至 SP7 後排程出現：
```
Could not load file or assembly 'System.Runtime, Version=8.0.0.0,
Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'. 系統找不到指定的檔案。
```

### 原因

SP7 用 **.NET 8.0**，Schedule.Core.exe 也是。若升級只更新了 Web 但忘了 Schedule.Core，舊版（例如 .NET 6）的 exe 找不到 net8 的 `System.Runtime`。

### 對策

升級時**同步重 publish `Schedule.Core`**：

```bash
dotnet publish Schedule.Core/Schedule.Core.csproj -c Release -f net8.0
```

並確認部署目標（`WebSitePath` 指向的網站目錄、`bin` 下的客製 dll）都是 net8 版本。

## 升級 checklist（方式 A）

每次升級 EEP 要跑過這個 checklist：

- [ ] `EEPWebClient.Core` publish + 部署到 IIS
- [ ] **`Schedule.Core` 同步重 publish**（勿漏）
- [ ] **客製 dll 同步放到 `Schedule.Core\bin\...\net8.0`**
- [ ] 重啟 Schedule.Core.exe（或 Windows Service）
- [ ] 檢查 SYS_SCHEDULE_LOG 確認新版有正常執行
- [ ] 若資料庫也有遷移，用設計介面的「測試」按鈕逐筆驗證排程

## 兩種方式的選擇建議

| 情境 | 建議 |
|------|------|
| **開發 / 測試階段** | **方式 B（強烈建議）** — exe 無法便利偵錯 |
| 單一伺服器、省部署 | **方式 B**（ScheduleHelper + BackgroundService）嵌入 Web |
| 大量/長時間排程任務，不想影響 Web | **方式 A（Schedule.Core.exe）** 獨立部署 |
| 有專用整合服務容器 | **方式 B**，自訂 host 包 ScheduleHelper |
| 跨伺服器負載平衡（Web 集群） | **方式 A 單機啟動**，避免集群每個 node 都跑一份重複執行 |
| 升級時要完全隔離 | **方式 A**，Web 升版不影響排程 |

> 若用方式 B 且 Web 是**多 instance / 集群**，必須只在**一台 node 啟動 ScheduleHelper**（或另外做 leader election），否則每個 node 都會跑 timer → 同一筆排程會執行 N 次。

## 常見陷阱

| 陷阱 | 說明 | 對策 |
|------|------|------|
| **🔴 以為裝好 Web 就會跑排程** | 公版 Web 不會自動啟動 ScheduleHelper（Startup.cs 啟動碼預設註解） | 部署 Schedule.Core.exe，或取消 Startup.cs 中的註解 |
| **排程設定建好了但沒動靜** | SYS_SCHEDULE 表有資料 ≠ 排程會跑；要有 process 在檢查才行 | 檢查是否有 Schedule.Core.exe 在執行，或 Web 有無自訂 StartSchedule |
| **🔴 用 Schedule.Core.exe 無法偵錯** | 獨立 process，斷點難接、熱重載無效、Console log 可視性差、例外堆疊被吃掉 | 開發/測試階段改用方式 B（Startup.cs 取消註解），驗證完再進正式 |
| **🔴 30 秒輪巡同分鐘觸發 2 次** | 官方規格，非 bug（來源：#478942） | ServerMethod 寫成冪等；或自行加鎖 |
| **🔴 客製 dll 排程跑不到** | Schedule.Core.exe 有獨立 bin 目錄，不共享 Web 的 dll | 把 dll 放 `Schedule.Core\bin\Debug\net8.0`；或改用方式 B 避開 |
| **🔴 Oracle 點選排程跳 ORA-00904** | 雙引號識別碼大小寫敏感，DBA 自動轉大寫會打掛 EEP 查詢（#479274 / #477328） | 用 `wwwroot/sql/oracle/systemTables.sql` 重建；確認在 System 資料庫 |
| **Schedule.Core.exe 卡死後重啟新的還是不跑** | 舊 process 沒被殺掉，兩個 exe 互相干擾（#480652） | 工作管理員確認舊 Schedule.Core.exe 結束後再重啟 |
| **手動測試 OK、自動排程不跑** | 沒啟動 exe 也沒取消 Startup.cs 註解（#482170） | 確認排程引擎已啟動 |
| **自動起單失敗、手動起單 OK** | 自動起單拿不到當前使用者，要在程式內顯式定義 user/url（#477391 / #476301 / #478706） | ServerMethod 內手動補 `this.client.user` 和 url 參數 |
| **登出 server 後 Schedule.Core 跟著停** | 互動 console 在使用者 session 下被殺（#482170） | 包成 Windows 服務、設開機自動啟動 |
| **Console 視窗被點選時排程暫停** | Windows console 被選取會進入「快速編輯」模式阻塞輸出（#476328） | 關閉 console 的快速編輯模式；或包成 Windows 服務避免 console |
| **「測試」按鈕 OK、自動排程失敗** | 測試走 Web process，排程走 exe process，dll 不一致 | 檢查 Schedule.Core 的 bin 目錄；或確認 exe 是否有啟動 |
| **interval 不是整點觸發** | interval 以「完成時間」起算，官方不改（來源：#481152 / #476305） | 改用 weekly + 多個 InvokeTime；或多筆排程 |
| **每月 31 日 ≠ 每月最後一天** | 2 月沒有 31 日 → 永遠不執行 | 設 28,29,30,31 + ServerMethod 判斷最後一天 |
| **升級 EEP 後排程壞：System.Runtime 找不到** | 只升 Web 沒同步升 Schedule.Core.exe | 每次升版同步 publish Schedule.Core（net8.0） |
| **雙軌邏輯分歧** | `Program.cs` 與 `ScheduleHelper.cs` 幾乎複製貼上，改一邊忘另一邊 | 改排程邏輯時兩個檔都要改 |
| **多 instance Web 啟用 ScheduleHelper** | 每個 node 都跑，任務被執行 N 次 | 只在一台 node 啟動，或轉用 Schedule.Core.exe |
| **`LastTime` 是 static** | 同 process 不要 `new ScheduleHelper()` 多次 | 啟動期只 new 一個 |
| **weekly/monthly 時間要剛好符合 HH:mm** | 內部比對的是 `startTime.ToString("HH:mm")`，只要當分鐘該到 | 30s timer + 同分鐘內會檢查兩次，第二次被 static `LastTime` 擋 |
| **interval 時窗寫錯** | `InvokeTime` 格式是 `"08:00,18:00"`，不是 JSON 陣列 | 設定時逗號分隔兩個 HH:mm |
| **`Console.ReadLine()` 阻塞** | 方式 A 用 Windows Service 包時會卡住 | 改用 `while(true) Thread.Sleep(Timeout.Infinite);` 或 `await Host.WaitForShutdown()` |
| **Disabled 不是布林欄位型別** | 比對用 `string.Compare(..., bool.TrueString, true) == 0`，即字串 `"True"` / `"true"` | 資料庫存入時確認是字串 True/False |
| **Schedule.Core 沒有檔案 log** | 只輸出 Console | Windows Service 用時 stdin/stdout 需重導向 |

## 相關討論區（完整彙整）

### 🔴 核心陷阱 / 官方確認為規格

| ID | 日期 | 主題 |
|----|------|------|
| [#478942](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=478942) | 2025-02-12 | Schedule.Core.exe 同一分鐘執行相同排程兩次 |
| [#481531](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481531) | 2025-09-22 | 排程重複觸發（每週一 9:30 會觸發兩次） |
| [#479186](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479186) | 2025-03-26 | 每小時排程實際發 2-3 次信 |
| [#481152](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481152) | 2025-08-04 | 間隔模式以完成時間起算 |

### DLL 隔離 / 測試 OK 自動失敗

| ID | 日期 | 主題 |
|----|------|------|
| [#481544](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481544) | 2025-09-23 | 排程測試 OK，自動執行日誌顯示異常 |
| [#480675](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480675) | 2025-06-03 | 排程執行不了 fs 模組 |
| [#480729](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480729) | 2025-06-11 | 排程呼叫 Server 問題 |
| [#482128](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482128) | 2025-12-29 | 排程設定後如何自動跑（未啟動 exe） |

### 設定 / 邊界條件

| ID | 日期 | 主題 |
|----|------|------|
| [#480387](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480387) | 2025-04-28 | 每月 31 日在 2 月的行為 |
| [#480651](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480651) | 2025-05-28 | IIS publish 後啟動排程 |
| [#481445](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481445) | 2025-09-10 | 設定排程自動起單 |
| [#481100](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481100) | 2025-07-29 | 自動起單 SubmitFlow 缺 url 參數 |

### Oracle / 部署環境

| ID | 日期 | 主題 |
|----|------|------|
| [#477328](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477328) | 2024-11-04 | **Oracle DB 排程設定錯誤 ORA-00904** — 雙引號大小寫陷阱、需在 System DB 執行 |
| [#479274](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479274) | 2025-04-09 | **點選排程跳 ORA-00904** — SYS_SCHEDULE_LOG 重建 SQL |
| [#480652](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480652) | 2025-05-29 | 排程突然停止（Schedule.Core.exe 卡死、舊 process 殘留）|
| [#482170](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482170) | 2026-01-07 | 手動測試 OK 自動不跑、Schedule.exe 包 Windows 服務 |

### 升級 / 部署

| ID | 日期 | 主題 |
|----|------|------|
| [#481157](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481157) | 2025-08-04 | 升級 SP6 後排程 Assembly 錯誤（net8.0） |
| [#482540](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482540) | 2026-04-07 | 資料庫移轉後排程異常（NullReferenceException） |
| [#481860](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481860) | 2025-11-20 | Schedule.Core 停止運作（建議更新新版 SP7） |
| [#481728](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481728) | 2025-10-31 | ScheduleCore.exe 寫入 LOG 異常 |

### 起單 / Flow 整合

| ID | 日期 | 主題 |
|----|------|------|
| [#480503](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480503) | 2025-05-08 | 自動起單反覆測試後失敗（FlowFlag 不可自行寫 InstanceID） |
| [#482048](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482048) | 2025-12-12 | 排程未執行（手動 OK、自動不跑） |
| [#479133](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479133) | 2025-03-18 | 排程送入流程 |
| [#481769](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481769) | 2025-11-08 | 排程測試「流程已經存在」 |
| [#477391](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477391) | 2024-11-13 | 手動 vs 自動起單差異（自動拿不到當前使用者，要在程式內定義 user/url） |
| [#476301](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=476301) | 2024-09 | 自動起單需顯式傳 url 參數 |
| [#478706](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478706) | 2025-01-09 | 排程啟單 userid 沒取到 |

### 其他

| ID | 日期 | 主題 |
|----|------|------|
| [#479301](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479301) | 2025-04-15 | 排程寫 90000 筆 USERS 影響 design |
| [#476305](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=476305) | 2024-09 | interval 規格說明（早期討論） |
| [#476328](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=476328) | 2024-10-28 | Console 視窗被選取時排程暫停（快速編輯模式陷阱） |
| [#481514](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481514) | 2025-09-18 | 是否可在 Client 網頁執行排程？（建議改 ServerMethod + DataForm OnApply） |
| [#478415](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478415) | 2024-11-28 | 排程執行時 server 取得當前模式值 |
| [#478933](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478933) | 2025-02-10 | 排程必須開著 Schedule.Core.exe 才會跑 |

## 參考檔案路徑

- 獨立 exe 進入點：`Schedule.Core/Program.cs`
- class library：`EEPServerTools.Core/Utility/ScheduleHelper.cs`
- 設計端管理：`EEPGlobal.Core/Provider/ScheduleProvider.cs`
- 系統表 SQL 定義：相關 `systemTables.sql`
- 相關資料表文件：
  - [SYS_SCHEDULE](../EEP%20Core系統資料表/SYS_SCHEDULE.md)
  - [SYS_SCHEDULE_LOG](../EEP%20Core系統資料表/SYS_SCHEDULE_LOG.md)
