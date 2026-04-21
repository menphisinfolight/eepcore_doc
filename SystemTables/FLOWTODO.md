# FLOWTODO

## 用途

**流程待辦事項表**（Flow To-Do）。

FLOWTODO 記錄當前等待處理的流程步驟，每筆記錄代表一個活動的待辦事項。包含處理者、發送者、權限標記（可編輯/可退回/可加簽等），並支援完整的代理人機制。是使用者「待辦清單」的資料來源。

### 使用場景

| 場景 | 說明 |
|------|------|
| **待辦查詢** | API.QueryToDo() 查詢當前使用者的待辦事項（含代理人的待辦） |
| **已辦查詢** | API.QueryHistory() 從 FLOWTODO 中篩選已出現在 FLOWHISTORY 的記錄 |
| **活動建立** | StandActivity / DetailFlowActivity / RootActivity 執行時寫入待辦 |
| **加簽管理** | 透過 PActivityID 追蹤加簽關係，LEFT JOIN 計算 PlusCount |
| **代理人機制** | CanAgent 欄位區分原始待辦與代理待辦，支援全流程/特定流程代理 |
| **流程分析** | FlowAnalysisProvider 統計待辦數量 |

### 關聯表

```
FLOWTODO ──(InstanceID)──> FLOWINSTANCE  （流程實例）
         ──(InstanceID)──> FLOWHISTORY   （簽核歷史）
         ──(InstanceID)──> FLOWNOTIFY    （通知）
         ──(InstanceID)──> FLOWCOMMENT   （簽核意見）
         ──(RoleID)─────> GROUPS         （角色/群組）
         ──(UserID)─────> USERS          （使用者）
         ──(PActivityID)─> FLOWTODO.ActivityID（加簽父活動，自我關聯）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowTodoHelper.cs` | 資料存取：Insert（3 種重載）/ Update / Delete / Select 系列 |
| `EEP.Flow/Instance.cs` | Reject() / Delete() 時刪除待辦 |
| `EEP.Flow/API.cs` | QueryToDo() / QueryHistory() / QueryAllToDo() / QueryDistinctToDo() |
| `EEP.Flow/Activities/StandActivity.cs` | 普通活動建立待辦，定義 Can* 權限屬性 |
| `EEP.Flow/Activities/DetailFlowActivity.cs` | 會簽活動建立待辦 |
| `EEP.Flow/Activities/RootActivity.cs` | 根活動（流程啟動）建立待辦 |
| `EEPGlobal.Core/Provider/FlowAnalysisProvider.cs` | 流程分析統計 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FlowID** | `nvarchar(50)` | NOT NULL | 流程定義編號 |
| **InstanceID** | `nvarchar(50)` | NOT NULL | 流程實例編號 |
| **FlowText** | `nvarchar(50)` | NOT NULL | 流程名稱 |
| **ActivityID** | `nvarchar(50)` | NOT NULL | 活動/步驟編號 |
| **ActivityText** | `nvarchar(50)` | NOT NULL | 活動/步驟名稱 |
| **RoleID** | `nvarchar(50)` | NULL | 處理角色編號（對應 GROUPS.GROUPID） |
| **RoleName** | `nvarchar(50)` | NULL | 處理角色名稱 |
| **UserID** | `nvarchar(50)` | NULL | 處理者帳號（對應 USERS.USERID） |
| **UserName** | `nvarchar(50)` | NULL | 處理者姓名 |
| **SenderID** | `nvarchar(50)` | NULL | 發送者帳號 |
| **SenderName** | `nvarchar(50)` | NULL | 發送者姓名，代理簽核時後綴 `(*)` |
| **Status** | `int` | NOT NULL | 流程狀態（見下方列舉） |
| **Datetime** | `datetime` | NOT NULL | 建立時間 |
| **ExpDatetime** | `datetime` | NULL | 預期完成時間 |
| **CanEdit** | `bit` | NULL | 是否可編輯表單（預設 true） |
| **CanPrint** | `bit` | NULL | 是否可列印（預設 false） |
| **CanReturn** | `bit` | NULL | 是否可退回（預設 true） |
| **CanAgent** | `bit` | NULL | 是否為代理待辦（預設 true） |
| **CanPlus** | `bit` | NULL | 是否可加簽（依 PlusMode 決定） |
| **CanReject** | `bit` | NULL | 是否可作廢（預設 false） |
| **Remark** | `nvarchar(2048)` | NULL | 備註/簽核意見 |
| **Parameter** | `nvarchar(max)` | NULL | 流程參數（JSON 格式） |
| **Duration** | `int` | NULL | 處理期限（天數） |
| **PActivityID** | `nvarchar(50)` | NULL | 父活動編號（加簽時指向原活動的 ActivityID） |
| **ProjectID** | `nvarchar(50)` | NULL | 所屬解決方案編號 |

### 跨資料庫差異

#### 欄位存在度

五個 DB `CREATE TABLE` 都含完整 25 欄（含 `CanEdit`/`CanPrint`/`CanReject`/`ExpDatetime`/`ProjectID` 等 SP7 新欄位）。

#### 型別對照

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `Datetime` / `ExpDatetime` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `Can*`（6 個 bool 欄位） | `bit` | `NUMBER(1)` | `bit` | `DECIMAL(1,0)` | `DECIMAL(1,0)` |
| `Parameter` | `nvarchar(max)` | `clob` | `text` | `NVARCHAR(8000)` ⚠️ | `LVARCHAR(8000)` ⚠️ |
| `Remark` | `nvarchar(2048)` | `varchar2(2048)` | `nvarchar(2048)` | `NVARCHAR(2048)` | `LVARCHAR(2048)` |

> ⚠️ **Parameter 欄位長度 DB2 / Informix 只有 8000 字元**（MSSQL/MySQL 為無上限，Oracle 是 CLOB 也無實質上限）。大型流程的 Parameter JSON 有機會超出，需注意。

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `CanEdit` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `CanReject` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `CanPrint` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `ExpDatetime` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `ProjectID` | ✅ | ❌ | ❌ | ❌ | ❌ |

5 個新欄位，Oracle / MySQL / DB2 / Informix CREATE TABLE 都已含但升級腳本**完全沒有 ALTER**。舊版升級到 SP7 時，四個非 MSSQL 的 DB 都要手動補欄位。

### Status 狀態列舉

| 值 | 狀態 | 說明 |
|----|------|------|
| 0 | Start | 啟動 |
| 1 | Submit | 審核 |
| 2 | Return | 退回 |
| 3 | Retake | 取回 |
| 5 | Plus | 加簽 |
| 6 | Transfer | 轉簽 |
| 7 | Reject | 作廢 |
| 8 | Prepare | 預啟動 |
| 9 | End | 結案 |
| 10 | Detail | 會簽 |
| 11 | Notify | 通知 |

### Can* 權限欄位預設值

| 欄位 | 預設值 | 來源屬性 | 說明 |
|------|--------|----------|------|
| CanEdit | true | StandActivity.CanEdit | 流程定義 XML 的 `CanEdit` 屬性 |
| CanPrint | false | StandActivity.CanPrint | 流程定義 XML 的 `CanPrint` 屬性 |
| CanReturn | true | StandActivity.CanReturn | 流程定義 XML 的 `CanReturn` 屬性 |
| CanAgent | true | StandActivity.CanAgent | 流程定義 XML 的 `CanAgent` 屬性 |
| CanPlus | 依 PlusMode | PlusMode != Disabled | PlusMode: Disabled / Enabled / Agent |
| CanReject | false | StandActivity.CanReject | 流程定義 XML 的 `CanReject` 屬性 |

---

## 索引

```
INDEX MAININDEX ON FlowToDo (InstanceID, ActivityID)
INDEX ROLEINDEX ON FlowToDo (RoleID, Datetime)
INDEX USERINDEX ON FlowToDo (UserID, Datetime)
```

---

## Insert 重載

| 重載 | 活動類型 | 說明 |
|------|----------|------|
| Insert(StandActivity) | 標準簽核活動 | 含完整權限標記與代理判斷 |
| Insert(DetailFlowActivity) | 會簽活動 | 會簽分支的待辦記錄 |
| Insert(RootActivity) | 根活動 | 流程啟動時的首筆待辦 |

---

## 代理人機制

代理人查詢分四個層面組合 OR 條件：

| 層面 | 條件 | 說明 |
|------|------|------|
| **群組待辦（被代理）** | `RoleID = role AND CanAgent != true` | 原始角色的待辦，排除已標記代理的項目 |
| **個人待辦（被代理）** | `UserID = user AND CanAgent != true` | 原始使用者的待辦 |
| **代理群組** | `RoleID = agentGroup AND CanAgent = true` | 代理人接收的群組待辦 |
| **代理個人** | `UserID = agentUser AND CanAgent = true` | 代理人接收的個人待辦 |

代理範圍：
- **全流程代理**：flowDesc 為空或 `*`，代理所有流程
- **特定流程代理**：flowDesc = `FlowID1,FlowID2`，僅代理指定流程

代理簽核時 SenderName 會加上 `(*)` 標記。IsAgented 屬性判斷當前執行者是否非原指定人員。

---

## PActivityID 與加簽

PActivityID 用於追蹤加簽（Plus）的父子關係：

- 加簽時，新建的待辦 PActivityID 指向原活動的 ActivityID
- 查詢待辦時，LEFT JOIN 計算每個活動的加簽數量（PlusCount）
- SelectPlus() 根據 PActivityID 查詢特定活動的所有加簽項
- DeletePlus() 根據 PActivityID + Status=Plus 刪除加簽記錄
- UpdateParameter() 同步更新加簽項的 Parameter

```sql
-- 加簽數量計算
SELECT todo.*, plus.PlusCount
FROM FlowToDo todo
LEFT JOIN (
    SELECT COUNT(FlowID) AS PlusCount, PActivityID, InstanceID
    FROM FlowToDo
    GROUP BY PActivityID, InstanceID
) plus ON plus.PActivityID = todo.ActivityID
     AND plus.InstanceID = todo.InstanceID
```

---

## 備註

- 此表**無主鍵**，同一 InstanceID 可有多筆記錄（多人同時待辦、加簽等場景）。
- 與 FLOWNOTIFY 結構相似，差異在於 FLOWTODO 是待辦（需簽核處理），FLOWNOTIFY 是知會（僅通知）。
- FLOWTODO 額外有 Status、CanEdit/CanReturn/CanAgent/CanPlus/CanReject、SenderID/SenderName、Duration、PActivityID 等欄位。
- QueryToDo(instanceID) 若查無待辦，會改查 FLOWHISTORY 的結案記錄作為回傳。
- 流程 Reject（作廢）或 Delete（刪除）時，會清除該實例的所有待辦記錄。
