# SYS_USERS_AGENT

## 用途

**使用者代理設定表**（User Agent Settings）。

SYS_USERS_AGENT 定義使用者的代理人設定，讓代理人在特定期間內代替原使用者處理流程簽核。與 SYS_ROLES_AGENT 為對偶表，結構相同但粒度不同（使用者 vs 角色）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **被代理使用者查詢** | `FlowProvider.GetAgentedUsers(user)` 查詢指定使用者被哪些代理人代理（PAR_AGENT != 'Y'） |
| **並行代理人查詢** | `FlowProvider.GetParAgentUserForUser(user, flowID)` 查詢 PAR_AGENT='Y' 的代理人 |
| **代理期間判斷** | 與 SYS_ROLES_AGENT 共用 AgentWheres 邏輯，以 START_DATE/TIME ~ END_DATE/TIME 判斷 |
| **代理流程過濾** | `IsAgentFlow()` 以 FLOW_DESC 判斷代理是否適用於指定流程 |
| **設計端管理** | `userAgent` infocommand + ucUserAgent：使用者代理設定的 CRUD；idsUserAgent 以 USERID→USER_ID 建立主從；idsUserAgent2 以 currentUser 為主表供個人設定用 |

### 關聯表

```
SYS_USERS_AGENT ──(USER_ID)──> USERS  （被代理者）
                ──(AGENT)───> USERS  （代理人帳號）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Adapter/Flow.cs` | `GetAgentedUsers()`：查被代理的使用者；`GetParAgentUserForUser()`：查並行代理人 |
| `SystemTable.Core/UserModule.cs` | `ucAgent_onBeforeUpdate()`：INSERT/UPDATE 前格式化日期時間（與 SYS_ROLES_AGENT 共用） |
| `EEPWebClient.Core/design/server/SystemTable.json` | `userAgent`：使用者代理 CRUD；`idsUserAgent`：管理端主從；`idsUserAgent2`：個人設定主從（currentUser 為主） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USER_ID** | `varchar(20)` PK | NOT NULL | 被代理者帳號（對應 USERS.USERID） |
| **AGENT** | `nvarchar(20)` PK | NOT NULL | 代理人帳號（對應 USERS.USERID） |
| **FLOW_DESC** | `nvarchar(40)` | NULL | 適用的流程名稱（空或 '*' = 全流程代理） |
| **START_DATE** | `nvarchar(8)` | NULL | 代理開始日期（格式：yyyyMMdd） |
| **START_TIME** | `nvarchar(6)` | NULL | 代理開始時間（格式：HHmmss） |
| **END_DATE** | `nvarchar(8)` | NULL | 代理結束日期 |
| **END_TIME** | `nvarchar(6)` | NULL | 代理結束時間 |
| **PAR_AGENT** | `nvarchar(4)` | NULL | 代理類型：空/'N' = 一般代理；'Y' = 並行代理 |
| **REMARK** | `nvarchar(254)` | NULL | 備註 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (USER_ID, AGENT)
```

### 索引

```
INDEX ROLEID ON SYS_USERS_AGENT (USER_ID)
```

（索引名稱 ROLEID 沿用舊名，實際索引欄位為 USER_ID）

---

## 備註

- 與 SYS_ROLES_AGENT 共用 `ucAgent_onBeforeUpdate` 事件處理和 `AgentWheres` 期間判斷邏輯。
- 設計端提供兩個 infodatasource：`idsUserAgent`（管理員視角，以 user 主表管理所有使用者的代理）和 `idsUserAgent2`（個人視角，以 currentUser 主表只管理自己的代理）。
- AGENT 欄位在此表為 `nvarchar(20)`（使用者帳號），而 SYS_ROLES_AGENT 的 AGENT 為 `nvarchar(50)`（群組/角色 ID），反映兩表的粒度差異。
