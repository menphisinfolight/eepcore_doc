# Flow

> `EEPServerTools.Core/Adapter/Flow.cs` — 2032 行

## 用途

**流程引擎介面卡**（Flow Adapter）。

Flow.cs 是 EEP Core 流程（簽核）系統的伺服器端核心，負責串接流程引擎 API（`EEP.Flow.API`）與資料庫。此檔案包含三個主要類別：

1. **`Flow`** — 流程操作入口，提供流程方法呼叫（送簽、退回、轉移等）、流程查詢、延遲自動簽核等功能。
2. **`FlowProvider`** — 流程資料提供者，實作多個流程引擎介面，負責使用者/群組/代理人/組織等資料查詢，以及流程事件（FlowFlag 更新）的處理。
3. **`FlowParser`** — 流程定義解析器，將 JSON 格式的流程設計圖轉換為 Activity 物件樹。

另外包含 `QueryParam`（查詢參數建構）以及 `DataSet`/`DataTable`/`DataRow` 的擴充方法。

## 實作介面

`FlowProvider` 實作以下介面（皆定義於 `EEP.Flow` 命名空間）：

| 介面 | 說明 |
|------|------|
| `IClientInfoProvider` | 提供當前使用者資訊（User、Groups、ProjectID、DBName 等） |
| `IAgentInfoProvider` | 提供代理人資訊（誰代理誰、角色代理、並行代理等） |
| `IOrgInfoProvider` | 提供組織資訊（主管查詢、組織層級等） |
| `IEventProvider` | 流程事件回呼（OnCreate、OnStart、OnSubmit、OnEnd 等） |
| `IMethodProvider` | 流程中呼叫自訂方法（Invoke） |

`FlowParser` 實作 `IParser` 介面。

`QueryParam` 實作 `IQueryParam` 介面。

## 核心方法 — Flow 類別

| 方法 | 說明 |
|------|------|
| `CallFlowMethod(param)` | 流程操作分派器，依 `param.method` 呼叫對應的 API 方法（Prepare / Start / Submit / Return / Retake / Reject / Delete / Transfer / Plus / Notify / Comment 等共 20+ 種操作） |
| `CallDelayMethod(param)` | 延遲自動簽核，查詢待簽核清單後逐筆呼叫 Submit；也支援 SendMail 補發通知 |
| `QueryFlow(param)` | 流程查詢分派器，依 `param.type` 查詢待辦（ToDo）、歷史（History）、知會（Notify）、警示（Warning）、結案（End）、留言（Comment）、明細（Detail）、流程定義（Defination）等 |
| `QueryData(param)` | 資料查詢分派器，查詢群組（Groups）、使用者（Users）、使用者群組（UserGroups）、群組使用者（GroupUsers）、使用者資訊（UserInfos） |
| `SaveToXml(param)` | 將流程 JSON 設計轉為 XML 格式並存檔 |
| `CreateFlowApi(param, transaction)` | 靜態方法，建立 `EEP.Flow.API` 實例，注入 FlowProvider、RedisLogProvider、FlowParser |
| `CreateDatabaseProvider(param, transaction)` | 靜態方法，依 `conn_type` 建立對應的資料庫 Provider（sql / mysql / oracle / db2 / informix） |

## 核心方法 — FlowProvider 類別

### 使用者與群組查詢

| 方法 | 說明 |
|------|------|
| `GetUserName(user)` | 從 USERS 表查詢使用者名稱 |
| `GetGroupName(group)` | 從 GROUPS 表查詢群組名稱 |
| `GetUsers(group)` | 取得群組下所有使用者 ID |
| `GetUserGroups()` | 取得當前使用者所屬的所有角色群組（ISROLE='Y'） |
| `GetGroupUsers(groups)` | 取得多個群組下的所有使用者（含 USERID、USERNAME、GROUPID） |
| `GetUsers(instanceID, pageSize, pageIndex, filter)` | 分頁查詢使用者清單，支援 SiteCode 過濾、DisNotifyGroups 排除、關鍵字篩選 |
| `GetGroups(instanceID, pageSize, pageIndex, filter)` | 分頁查詢角色群組清單，同樣支援 SiteCode、DisNotifyGroups、關鍵字篩選 |
| `GetUserInfos(group, user, canAgent, flowID)` | 綜合查詢：取得指定群組/使用者的最終簽核人清單，考慮代理人設定，回傳包含 UserType（user/agent）欄位 |

### 代理人查詢

| 方法 | 說明 |
|------|------|
| `GetAgentedGroups(group)` | 查詢指定群組是否被代理（SYS_ROLES_AGENT，PAR_AGENT != 'Y'），回傳 `List<KeyValuePair<ROLE_ID, FLOW_DESC>>` |
| `GetAgentedUsers(user)` | 查詢指定使用者是否被代理（SYS_USERS_AGENT，PAR_AGENT != 'Y'） |
| `GetAgentGroups(user)` | 查詢指定使用者代理了哪些群組（AGENT = user） |
| `GetAgentUsers(user)` | 查詢指定使用者代理了哪些使用者（AGENT = user） |
| `GetParAgentUserForGroup(group, flowID)` | 取得群組的並行代理人（PAR_AGENT = 'Y'），依 FLOW_DESC 過濾流程 |
| `GetParAgentUserForUser(user, flowID)` | 取得使用者的並行代理人（PAR_AGENT = 'Y'），依 FLOW_DESC 過濾流程 |

### 組織查詢

| 方法 | 說明 |
|------|------|
| `GetManagerID(roleID, levelNo, orgKind)` | 依角色 ID 向上遞迴查詢指定層級的主管（SYS_ORG / SYS_ORGROLES），回傳 ORG_MAN |
| `GetOrgLevelName(levelNo)` | 從 SYS_ORGLEVEL 查詢組織層級名稱 |

### 流程事件（IEventProvider）

| 事件方法 | FlowFlag 值 | 說明 |
|----------|------------|------|
| `OnCreate` | — | 設定 Instance 的 Database 和 ProjectID，檢查 FlowFlag 防止重複送簽 |
| `OnPrepare` | `P` | 暫存（Prepare）狀態 |
| `OnStart` | `N` | 啟動流程 |
| `OnSubmit` | `N` | 送簽 |
| `OnRetake` | `P`（若退回至起點） | 抽回 |
| `OnReject` | `X` | 駁回 |
| `OnDelete` | 空字串 | 刪除流程，清除 FlowFlag |
| `OnEnd` | `Z` | 結案 |

### 其他

| 方法 | 說明 |
|------|------|
| `Invoke(instance, methodName)` | 流程中呼叫自訂方法，支援 `PROC.方法名`（透過 Processor）或 `模組名.方法名`（透過 DataModule 反射） |
| `RefreshParameter(instance)` | 重新從資料庫讀取表單資料，更新 Instance.Parameter（用於送簽時取得最新欄位值） |
| `CheckFlowFlag(instance)` | 檢查表單的 FlowFlag 欄位，若已有流程則拋出 `flowExist`；若為 `Z:` 開頭則允許重新送簽 |
| `UpdateFlowFlag(instance, flowflag)` | 更新表單的 FlowFlag 欄位為 `{flag}:{InstanceID}` 格式 |
| `SelectUserTable(tableName, parameters)` | 從使用者資料庫查詢資料表 |

## FlowFlag 狀態機

```
（無值）→ P（暫存）→ N（簽核中）→ Z（結案）
                  ↘ X（駁回）
Z:InstanceID → 允許重新啟動流程
```

FlowFlag 寫入格式為 `{狀態}:{InstanceID}`，存放於表單資料表的 FlowFlag 欄位。

## 代理人時間判斷（AgentWheres）

代理人查詢統一使用 `AgentWheres` 屬性產生時間範圍條件：

- `START_DATE + START_TIME <= 當前時間（yyyyMMddHHmmss）`
- `END_DATE + END_TIME >= 當前時間（yyyyMMddHHmmss）`
- 空值或 NULL 視為不限

## FlowParser — 流程定義解析

`FlowParser.ToActivity(text, json)` 將前端流程設計器輸出的 JSON 陣列轉為 `RootActivity` 物件樹：

1. 分離 `line`（連線）、`activity`（活動節點）、`flow`（流程屬性）
2. 依 line 的 source/target 排序活動順序
3. 透過反射建立對應的 Activity 子類別（如 `ApproveActivity`、`CompositedActivity` 等）
4. 遞迴處理子活動和 ApproveRights

## QueryParam — 查詢參數

支援的查詢欄位：

| 參數名 | 查詢方式 |
|--------|---------|
| `FlowText` | LIKE（前綴匹配） |
| `ActivityText` | LIKE（模糊匹配） |
| `Parameter` | LIKE（模糊匹配） |
| `DatetimeFrom` / `DatetimeTo` | 日期範圍（>= / <） |
| `Sender` | SenderID 或 SenderName LIKE |
| `User` | UserID 或 UserName LIKE |
| `Role` | RoleID 或 RoleName LIKE |
| `IsRead` / `IsFinish` | 精確比對（布林） |
| `Level` / `Type` | 精確比對 |
| `WarningText` | Subject 或 Description LIKE |
| `FlowStatus` | end → Status=9、reject → Status=7 |
| `StartByMe` | Status IN (0,8) |

若系統啟用 `IsBySolution`，會自動加上 ProjectID 過濾。

## 備註

- Flow 類別本身不實作介面，而是作為入口負責建立 API 和分派呼叫；實際的資料提供由 `FlowProvider` 負責。
- `CreateDbCommand` 內含大量資料庫特殊處理：Oracle 使用字串替換參數、Informix 使用 ODBC 參數、STMAS400/PISTHOST 使用 `?` 佔位符。
- 代理人分為兩種：一般代理（PAR_AGENT != 'Y'）會完全取代原簽核人，並行代理（PAR_AGENT = 'Y'）則是額外加入簽核。
- `GetInstance` 使用 `BinaryFormatter` 反序列化流程實例（`FlowInstance.State` 欄位），搭配 `FlowInstanceBinder` 處理型別綁定。
- 檔案末尾的 `DataSetExtension`、`DataTableExtension`、`DataRowExtension` 是定義在 `System.Data` 命名空間下的擴充方法，將 DataTable/DataRow 轉為 JObject/JArray。
- 許多舊版代理人查詢方法已被註解掉，目前使用的版本支援時間精確到秒（yyyyMMddHHmmss）以及 FLOW_DESC 流程過濾。
