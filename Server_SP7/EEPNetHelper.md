# EEPNetHelper

> `EEPServerTools.Core/Utility/EEPNetHelper.cs` — 981 行

## 用途

**EEP.NET 整合工具**（EEP.NET Integration Helper）。

EEPNetHelper 是 EEP Core 與舊版 EEP.NET 系統之間的橋樑，負責 SSO 單一登入（Single Sign-On）以及流程引擎的跨平台通訊。檔案包含兩個類別：

- **EEPNetHelper**（基底類別）：管理 `appsettings.json` 設定、SSO 金鑰取得、HTTP POST 通訊。
- **EEPNetFlowHelper**（衍生類別）：實作 EEP.NET 流程引擎的查詢與操作（待辦、歷程、知會、結案、送簽、退回等）。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `ClientInfo` | ClientInfo | 當前使用者連線資訊 |
| `IsEEPNETFlow` | static bool | FlowSource 是否為 `"EEPNET"` |
| `IsBothFlow` | static bool | FlowSource 是否為 `"BOTH"`（同時使用 EEP Core 與 EEP.NET 流程） |
| `IsFlowWarning` | static bool | 是否啟用流程警告（讀取 ConfigHelper） |
| `IsBySolution` | static bool | 流程是否依 Solution 區分 |
| `ServiceUrl` | static string | EEP.NET 服務基底 URL（`appsettings.json` 的 `EEPNETUrl`） |
| `SSOKey` | string | SSO 金鑰（首次取得後快取在 ClientInfo 中） |

## 核心方法 — EEPNetHelper（基底）

| 方法 | 參數 | 回傳 | 說明 |
|------|------|------|------|
| `GetSSOKey()` | — | string | 呼叫 `SingleSignOn.asmx/LogOn` 取得 SSO 金鑰（傳入 userId、password、dataBase、solution） |
| `GetSSOUrl(id)` | string id | string | 組合 SSO URL，以 `MenuID` 為參數 |
| `GetResponseStr(url, parameters)` | string, Dictionary | string | 發送 HTTP POST 並回傳回應字串 |
| `GetResponse(url, parameters)` | string, Dictionary | HttpWebResponse | 底層 HTTP POST 實作（URL-encoded form data） |

## 核心方法 — EEPNetFlowHelper（流程操作）

### 流程查詢（QueryFlow）

透過 `QueryFlow(IFormCollection param)` 統一入口，依 `type` 參數分派：

| type | 方法 | 呼叫的 EEP.NET 服務 | 說明 |
|------|------|------|------|
| `ToDo` | `QueryToDo` | `GetToDoListForAllAllPlatform` | 待辦清單 |
| `History` | `QueryHistory` | `GetToDoHisForAllAllPlatform` | 已辦歷程 |
| `Notify` | `QueryNotify` | `GetNotifyForAllAllPlatform` | 知會清單 |
| `End` | `QueryEnd` | `GetEndForAllAllPlatform` | 結案清單 |
| `Detail` | `QueryDetail` | `GetCommentForAllAllPlatform` | 流程明細（簽核意見） |
| `Comment` | `QueryComment` | — | 評論（目前回傳空 DataTable） |
| `CurrentComment` | `QueryCurrentComment` | — | 當前評論（目前回傳空 DataTable） |
| `PrevivousActivities` | `GetPrevivousActivities` | `GetFLPathListForAllAllPlatform` | 取得可退回的前步驟清單 |

### 流程操作（CallFlowMethod）

透過 `CallFlowMethod(IFormCollection param)` 統一入口，依 `method` 參數分派：

| method | 呼叫的 EEP.NET 服務 | 說明 |
|--------|------|------|
| `Submit` | `ApproveForAllPlatform` | 送簽（核准並送往下一關） |
| `Return` | `ReturnForAllPlatform` / `ReturnToForAllPlatform` | 退回（指定退回對象時用 ReturnTo） |
| `Retake` | `RetakeForAllPlatform` | 抽回（收回已送出的表單） |
| `Notify` | `NotifyForAllPlatform` | 知會（傳送給指定使用者或角色） |
| `DeleteNotify` | `DeleteNotifyForAllPlatform` | 刪除知會 |
| `Preview` | `PreviewForAllPlatform` | 預覽流程路徑 |

### 檔案操作

| 方法 | 參數 | 回傳 | 說明 |
|------|------|------|------|
| `UploadFile` | string fileName, byte[] content | void | 上傳附件至 EEP.NET（Base64 編碼） |
| `DownloadFile` | string fileName | byte[] | 從 EEP.NET 下載附件（Base64 解碼） |

## 關鍵邏輯

### SSO 機制

1. 首次存取 `SSOKey` 時，以使用者帳號/密碼/資料庫/方案呼叫 EEP.NET 的 `SingleSignOn.asmx/LogOn`。
2. 回傳 XML 中取出 SSO 金鑰，快取至 `ClientInfo["SSOKey"]`，後續呼叫不再重新登入。
3. 所有 EEP.NET 服務呼叫都以 `securityKey`（即 SSOKey）做身份驗證。

### 流程查詢的資料轉換

EEP.NET 回傳的 XML DataSet 欄位名稱（如 `FLOW_ID`、`LISTID`、`D_STEP_ID`）與 EEP Core 前端期望的欄位不同。每個 `Convert*Table` 方法負責：
- 建立標準化的 DataTable 結構（FlowID、InstanceID、FlowText 等）。
- 逐列將 EEP.NET 欄位映射到 EEP Core 欄位。
- 處理格式轉換：`FormatStatus`（狀態碼 N/A/X/P/Z → 數字 1~9）、`FormatDatetime`、`FormatExpDatetime`（到期時間，依時間單位 Hour/Day 計算）。

### FilterTable — 查詢與分頁

`FilterTable` 統一處理查詢過濾和分頁邏輯：
- 支援的篩選條件：`FlowText`（精確比對）、`Parameter`（模糊搜尋）、`DatetimeFrom`/`DatetimeTo`（日期範圍）、`Sender`/`User`/`Role`（ID 或名稱模糊搜尋）、`FlowStatus`（end=9、reject=7）。
- 分頁：以 `rows`（每頁筆數）和 `page`（頁碼，1-based）參數實作。

### 狀態碼對照

| EEP.NET 狀態 | EEP Core 狀態碼 | 意義 |
|------|------|------|
| N | 1 | 新案 |
| NR | 2 | 新案（退回） |
| NF | 3 | 新案（轉送） |
| A / AA | 5 | 核准 |
| X | 7 | 駁回 |
| P | 8 | 加簽 |
| Z | 9 | 結案 |

## 備註

- 此檔案**不包含**通知管道功能（LINE、Teams、釘釘、SMS、Email 等），通知相關邏輯位於其他 Helper（如 FlowHelper）。
- `Configuration` 為 static 欄位，`appsettings.json` 只讀取一次後快取，修改設定需重啟服務。
- HTTP 通訊使用 `HttpWebRequest`（非 HttpClient），為舊式同步呼叫方式。
- `MailAddress` 屬性組合了 EEP.NET 的 `RWDMainFlowPage.aspx` URL 樣板，用於 Submit/Return 時傳送郵件通知連結。
- `QueryComment` 和 `QueryCurrentComment` 目前回傳空 DataTable，尚未實作。
- `FormatCanRetake` 判斷條件：狀態為新案（1）且送出者為當前使用者時可抽回。
