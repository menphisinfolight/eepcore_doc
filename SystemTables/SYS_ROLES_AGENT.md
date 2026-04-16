# SYS_ROLES_AGENT

## 用途

**角色代理設定表**（Role Agent Settings）。

SYS_ROLES_AGENT 定義角色的代理人設定，讓代理人在特定期間內代替角色成員處理流程簽核。支援設定代理期間（含時間）、代理流程範圍、代理層級（一般代理/並行代理）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **被代理群組查詢** | `FlowProvider.GetAgentedGroups(group)` 查詢指定角色被哪些代理人代理（PAR_AGENT != 'Y'），用於排除被代理的待辦 |
| **並行代理人查詢** | `FlowProvider.GetParAgentUserForGroup(group, flowID)` 查詢 PAR_AGENT='Y' 的代理人，將待辦同時發給代理人 |
| **代理期間判斷** | AgentWheres 屬性：以 START_DATE+START_TIME 和 END_DATE+END_TIME 判斷代理是否在有效期間內 |
| **代理流程過濾** | `IsAgentFlow()` 以 FLOW_DESC 判斷代理是否適用於指定流程（空值或 '*' 代表全流程代理） |
| **設計端管理** | `roleAgent` infocommand + ucRoleAgent：角色代理設定的 CRUD，透過 idsRoleAgent 以 GROUPID→ROLE_ID 建立主從關係 |

### 關聯表

```
SYS_ROLES_AGENT ──(ROLE_ID)──> GROUPS  （角色，ISROLE='Y'）
                ──(AGENT)───> GROUPS   （代理角色/群組）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Adapter/Flow.cs` | `GetAgentedGroups()`：查被代理的角色；`GetParAgentUserForGroup()`：查並行代理人 |
| `SystemTable.Core/UserModule.cs` | `ucAgent_onBeforeUpdate()`：INSERT/UPDATE 前格式化日期時間（去除 `/` `-` `:`） |
| `EEPWebClient.Core/design/server/SystemTable.json` | `roleAgent`：角色代理 CRUD；`idsRoleAgent`：以 role 為主表的主從關係 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ROLE_ID** | `varchar(50)` PK | NOT NULL | 被代理的角色識別碼（對應 GROUPS.GROUPID） |
| **AGENT** | `nvarchar(50)` PK | NOT NULL | 代理人（群組/角色 ID） |
| **FLOW_DESC** | `nvarchar(40)` | NULL | 適用的流程名稱（空或 '*' = 全流程代理） |
| **START_DATE** | `nvarchar(8)` | NULL | 代理開始日期（格式：yyyyMMdd，空 = 無限制） |
| **START_TIME** | `nvarchar(6)` | NULL | 代理開始時間（格式：HHmmss，空 = 000000） |
| **END_DATE** | `nvarchar(8)` | NULL | 代理結束日期（空 = 無限制） |
| **END_TIME** | `nvarchar(6)` | NULL | 代理結束時間（空 = 235959） |
| **PAR_AGENT** | `nvarchar(4)` | NULL | 代理類型：空/'N' = 一般代理（排除原待辦）；'Y' = 並行代理（原待辦和代理同時收到） |
| **REMARK** | `nvarchar(254)` | NULL | 備註 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (ROLE_ID, AGENT)
```

### 索引

```
INDEX ROLEID ON SYS_ROLES_AGENT (ROLE_ID)
```

---

## 代理判斷邏輯

```
AgentWheres（期間判斷）：
  (START_DATE 為空 OR ISNULL(START_DATE,'') + ISNULL(START_TIME,'000000') <= @now)
  AND
  (END_DATE 為空 OR ISNULL(END_DATE,'') + ISNULL(END_TIME,'235959') >= @now)

GetAgentedGroups（被代理查詢）：
  WHERE ROLE_ID = @group AND PAR_AGENT != 'Y' AND 期間有效
  → 回傳被代理的 ROLE_ID 和 FLOW_DESC

GetParAgentUserForGroup（並行代理）：
  WHERE ROLE_ID = @group AND PAR_AGENT = 'Y' AND 期間有效
  → 過濾 FLOW_DESC 後回傳 AGENT 列表
```

---

## 備註

- PAR_AGENT 決定代理模式：一般代理時，原角色的待辦會被標記為被代理（CanAgent），代理人在待辦清單中看到被代理的項目。並行代理時，原角色和代理人同時收到待辦。
- 日期時間使用字串格式（yyyyMMdd / HHmmss），`ucAgent_onBeforeUpdate` 會自動去除 `/`、`-`、`:` 分隔符號。
- 舊版的 IsUserAgented / FilterAgentedGroups / GetAgentGroups / GetAgentUsers 方法已被註解掉，替換為新版的含時間判斷版本。
- 與 SYS_USERS_AGENT 為對偶表：本表是角色級代理，SYS_USERS_AGENT 是使用者級代理。
