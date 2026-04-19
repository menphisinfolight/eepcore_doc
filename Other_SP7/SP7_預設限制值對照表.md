# SP7 預設限制值對照表

SP7 有一批預設的上限值與 Timeout——筆數、大小、逾時等——散落在不同原始碼、config 與元件裡。這份文件把這類常用預設值集中列表，方便客戶或同仁需要調整時能快速定位原始碼與調整方式。

> 之後如需補充新的預設值，就在這裡加一行；不要把同一段寫進每個元件 doc。

## 對照表

### 資料查詢 / 匯出

| # | 項目 | 預設值 | 調整方式 | 位置 |
|---|------|--------|---------|------|
| 1 | Excel 匯出筆數 | **1000 筆** | 呼叫 `ExportExcel` 時在 `ExportExcelOptions.PageSize` 傳入較大值（> 0 才生效）；或改底層預設 | `EEPServerTools.Core/Utility/ParserHelper.cs` L793 |
| 2 | InfoCommand Command Timeout | **30 秒** | 在 InfoCommand JSON 設 `CommandTimeout`；或 `helper.CommandTimeout = 180;` | `EEPServerTools.Core/Components/InfoCommand.cs` L71；套用於 `DataModule.cs` L625/637/656/716 |
| 3 | DatabaseHelper Command Timeout | **30 秒** | 屬性直接指派 | `EEPBase.Core/Utility/DatabaseHelper.cs` L90；`EEP.Flow/Sqls/SqlHelper.cs` L44 |
| 4 | EEP.Flow SQL 連線 Timeout | **5 秒** | 連線字串帶入 `Connection Timeout=N` 即會沿用自訂值 | `EEP.Flow/Interfaces/Database.cs` L225 |
| 5 | LDAP search page size | **1000** | 改程式 | `EEPBase.Core/Utility/LDAPHelper.cs` L43；`EEPAdapater/LDAP.cs` L62 |
| 6 | SecurityProvider 安全規則查詢上限 | **10000 筆** | 改程式（需評估效能） | `EEPGlobal.Core/Provider/SecurityProvider.cs` L47、L56 |

### 檔案與請求大小

| # | 項目 | 預設值 | 調整方式 | 位置 |
|---|------|--------|---------|------|
| 7 | IIS 請求內容最大 | **1 GB**（1073741824 bytes） | 改 `web.config` `<requestLimits maxAllowedContentLength="..." />` | `EEPWebClient.Core/web.config` L14 |
| 8 | ASP.NET Form 欄位數量 / 值長度 | **`int.MaxValue`**（SP7 主動放寬） | 已解除 ASP.NET 預設 1024 欄位 / 4MB 單值限制；正常無需再調整 | `EEPWebClient.Core/Startup.cs` L70-71 |
| 9 | FileProvider 上傳 body 限制 | **無限**（`long.MaxValue`，SP7 主動放寬） | 實際上限仍受 IIS 層的 1 GB 規範（第 7 條） | `EEPGlobal.Core/Provider/FileProvider.cs` L477 |

### Session / 登入 / Web 伺服器

| # | 項目 | 預設值 | 調整方式 | 位置 |
|---|------|--------|---------|------|
| 10 | ASP.NET Core Session `IdleTimeout`（sliding） | **20 分鐘**（framework 預設，SP7 未覆寫） | 在 `AddSession(options => options.IdleTimeout = TimeSpan.FromMinutes(60))` 覆寫 | `EEPWebClient.Core/Startup.cs` L36、L47 未設 `IdleTimeout` |
| 11 | SP7 前端 `refreshLogon` 定期 ping | 設計端 60 秒 / 執行端 `clientInfo.refreshInterval` 分鐘 / Webview 5 分 | 執行端在設計端 Global 設定 `pushmsgrefreshInterval`（0 = 停用） | 設計端：`jquery.infolight.design.js` L9；執行端：`bootstrap.infolight.main.js` L161/164/485 |
| 12 | SessionInfos 閒置清除閾值（SP7 自家，非 ASP.NET Core Session） | **30 分鐘**；清理 Timer 每 60 分跑一次 | 改程式 | `EEPBase.Core/ClientInfo.cs` L236（閾值）、L149（Timer 間隔） |
| 13 | Kestrel 預設 timeouts（SP7 未覆寫） | KeepAlive 130 秒 / RequestHeaders 30 秒 | 在 `Program.cs` 啟用 `ConfigureKestrel(...)`；目前整段被註解（L24-33） | `EEPWebClient.Core/Program.cs` L24-33 |

### 外部服務

| # | 項目 | 預設值 | 調整方式 | 位置 |
|---|------|--------|---------|------|
| 14 | ChatProvider（LLM 呼叫）HTTP Timeout | **10 分鐘** | 改程式 | `EEPGlobal.Core/Provider/ChatProvider.cs` L278 |

---

## 細節說明

### 1. Excel 匯出筆數（1000）

**常見情境**：客戶按「匯出 Excel」後拿到的檔案筆數固定 ≤ 1000，但查詢清單明明更多。

**實作細節**：`ParserHelper.ExportExcel` L787-793 的邏輯：

```csharp
if (options.PageSize > 0)
{
    queryOptions.PageSize = options.PageSize;   // 呼叫端有傳，就聽它的
}
else
{
    queryOptions.PageSize = 1000;               // fallback = 1000
    ...
}
```

→ **如果呼叫端沒指定 PageSize**，就會硬限 1000。這條限制是**可以用參數覆寫**的，不一定要改底層。

**建議做法**（依優先序）：

1. 呼叫 `ExportExcel(id, new ExportExcelOptions { PageSize = 10000, ... })` — 最乾淨
2. 若該功能無法改呼叫端，才動底層 `ParserHelper.cs` L793；但要注意 **IIS 部署需重新 `dotnet publish`** 才會生效（這條在《IIS 發佈》文件已有通則，此處不重複）

### 2–3. Command Timeout（30 秒）

**常見情境**：複雜 SQL 或大批資料查詢拋 `timeout expired` 例外。

**套用順序**：

```
InfoCommand JSON 的 CommandTimeout
    → DataModule 傳給 DbHelper
    → DbHelper.CommandTimeout 套到每次 cmd 執行
```

因此**優先在 InfoCommand JSON 設定**（如 `"CommandTimeout": 180`），不需改 C#。若是直接 `new DatabaseHelper()` 手動建 helper，則 `helper.CommandTimeout = 180;`。

### 4. EEP.Flow SQL 連線 Timeout（5 秒）

`FlowSqlDatabase` 建構時若連線字串**沒有**指定 `Connection Timeout`，會自動附加 `;Connection Timeout=5`。若環境網路抖動導致 flow 拋出「連線逾時」，可在連線字串中自行指定較大值（例如 `Connection Timeout=30`）繞過預設。

注意這個 5 秒是 **「建立連線」的 timeout**，與第 2~3 條的 **「執行命令」的 timeout** 是不同層。

### 5. LDAP page size（1000）

**常見情境**：使用 AD 同步時，成員數超過 1000 的群組抓不全。

LDAP server 端本身就有 MaxPageSize（Windows AD 預設也是 1000），client 端若設更大也會被 server 截。通常要搭配 paged cookie 才能真正抓完，EEP 現階段用固定 1000。

### 6. SecurityProvider 安全規則查詢（10000）

`.security` 檔的規則載入時，若單一資料表有超過 10000 筆規則，超過的部分不會被讀入。極罕見，一般專案難以觸及。

### 7. IIS 請求內容最大（1 GB）

**常見情境**：超大檔上傳時，IIS 回 **413 Payload Too Large**，此時連 ASP.NET 都還沒接到請求。

**注意**：這是 **IIS 層**的限制，跟 ASP.NET 的 `MaxRequestBodySize` 是兩層（Kestrel 裡那條在 `Startup.cs` L64 目前註解掉）。IIS 託管時以 `web.config` 為準。

### 8. ASP.NET Form 欄位 / 長度（int.MaxValue）

SP7 主動在 `Startup.cs` 呼叫：

```csharp
services.Configure<FormOptions>(options =>
{
    options.ValueCountLimit = int.MaxValue;
    options.ValueLengthLimit = int.MaxValue;
});
```

解除 ASP.NET 預設的 1024 欄位 / 4MB 單欄位長度限制。因此大型 Datagrid 批次送回、含很多欄位的表單，不會卡在 framework 層。

### 9. FileProvider 上傳（無限）

SP7 已主動在該 endpoint 加 `[RequestFormLimits(MultipartBodyLengthLimit = long.MaxValue)]`，放寬 ASP.NET 預設的 128MB 表單限制。**實際上限仍受 IIS 層的 1 GB 規範**（見第 7 條）。

### 10 & 11. Session IdleTimeout + refreshLogon ping（為何 session 感覺「永遠不 timeout」）

這是容易誤解的一組：

- **框架層**：ASP.NET Core 預設 `SessionOptions.IdleTimeout = 20 分鐘`，而 SP7 的 `Startup.cs` 呼叫 `AddSession()` 時**沒有覆寫** `IdleTimeout`，所以吃預設值。
- **重點**：`IdleTimeout` 是 **sliding window**——**任何請求打到伺服器就會重置 timer**，不是固定 20 分鐘就失效。
- **SP7 前端主動 ping**：
  - 設計端（Design IDE）：`jquery.infolight.design.js` L9 每 **60 秒** `refreshLogon` 一次（固定）
  - 執行端（RWD / 主站）：`bootstrap.infolight.main.js` L161 依 `clientInfo.refreshInterval` 分鐘 ping；使用者在設計端 Global 設定 `pushmsgrefreshInterval`（0 = 停用）
  - Webview（手機 App）：L164 固定 **5 分鐘** ping
  - `refreshLogon` 實作在 L485：`POST /logon mode=refreshLogon`，回 `timeout` 就登出並 reload，否則 session 時鐘重置

**結論**：只要瀏覽器開著、頁面沒關、`refreshInterval` 設小於 20 分鐘，**Session 永遠不會 timeout**。這就是為何使用者實務上感覺「一小時以上都沒逾時」。

**真正會 timeout 的情境**：
1. 瀏覽器完全關閉（Session cookie 預設 browser-scoped，關閉即失效）
2. `refreshInterval` 設為 0（停用）或大於 20 分鐘，且超過 20 分完全沒互動
3. 手機 App webview 背景超過 5 分鐘沒被喚醒 + 再超過 20 分鐘沒 ping
4. 網路中斷超過 20 分鐘

### 12. SessionInfos 閒置清除（SP7 自家，30 分鐘）

`EEPBase.Core/ClientInfo.cs` 裡有一個 SP7 **自己維護**的 in-memory `SessionInfos` dictionary，用來追蹤目前線上的 ClientInfo。L149 的 Timer 每 60 分跑一次 `RemoveExpired()`，L236 判斷 `span.TotalMinutes > 30.0` 就清掉。

**和第 10 條是兩個不同系統**：ASP.NET Core Session 管 session cookie 的生命週期；`SessionInfos` 是 SP7 自用的線上使用者清單。兩者 timeout 不同、判斷時機也不同。

### 13. Kestrel 預設 timeouts（130s / 30s）

`EEPWebClient.Core/Program.cs` L24-33 的 `UseKestrel(...)` 整段被註解，所以吃 Kestrel framework 預設：

- `KeepAliveTimeout` = 130 秒
- `RequestHeadersTimeout` = 30 秒
- `MaxConcurrentConnections` / `MaxConcurrentUpgradedConnections` = 無上限
- `MaxRequestBodySize` = 30 MB（但 IIS 託管時看 `web.config` 的 1 GB）

**IIS 託管時影響小**（IIS 有自己一層）；**直接 `dotnet run` 時會遇到**。若需調整，在 `Program.cs` 啟用 `ConfigureKestrel(...)`。

### 14. ChatProvider Timeout（10 分）

**常見情境**：呼叫 GPT/Claude 長回應或 streaming 一開始就被中斷。10 分鐘通常夠用，除非跑 long-thinking 的 model。
