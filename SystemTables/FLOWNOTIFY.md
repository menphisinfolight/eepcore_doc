# FLOWNOTIFY

## 用途

**流程通知表**（Flow Notification）。

FLOWNOTIFY 儲存流程的知會/通知記錄，當流程進行到某步驟時，通知相關人員（不需簽核，僅告知）。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **知會活動** | 流程設計中的 NotifyActivity 節點執行時，自動寫入通知記錄 |
| **通知查詢** | 使用者透過「流程通知」功能查看所有知會自己的事項 |
| **批次通知** | 支援發送給多個角色/使用者（如 AllUsers 模式） |
| **參照欄位通知** | 可根據表單欄位值動態決定通知對象（RefRole/RefUser） |

### 與 FLOWTODO 的差異

| 項目 | FLOWNOTIFY | FLOWTODO |
|------|------------|----------|
| **處理方式** | 僅知會，不需處理 | 待辦事項，需簽核處理 |
| **狀態欄位** | 無狀態欄位 | 有 Status 欄位（未處理/已處理） |
| **代理人機制** | 不支援代理人 | 支援代理人（CanAgent） |
| **加簽功能** | 不支援 | 支援加簽（Plus） |

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **FlowID** | `nvarchar(50)` | 流程定義 ID |
| **InstanceID** | `nvarchar(50)` | 流程實例 ID |
| **FlowText** | `nvarchar(50)` | 流程名稱 |
| **ActivityID** | `nvarchar(50)` | 活動/步驟 ID |
| **ActivityText** | `nvarchar(50)` | 活動/步驟名稱 |
| **RoleID** | `nvarchar(50)` | 通知對象角色 ID |
| **RoleName** | `nvarchar(50)` | 通知對象角色名稱 |
| **UserID** | `nvarchar(50)` | 通知對象帳號 |
| **UserName** | `nvarchar(50)` | 通知對象姓名 |
| **SenderID** | `nvarchar(50)` | 發送者帳號 |
| **SenderName** | `nvarchar(50)` | 發送者姓名 |
| **Datetime** | `datetime` | 通知時間 |
| **CanPrint** | `bit` | 是否可列印 |
| **Remark** | `nvarchar(2048)` | 備註（流程主旨/說明） |
| **Parameter** | `nvarchar(max)` | 參數（流程傳遞的參數資料） |
| **ProjectID** | `nvarchar(50)` | 專案 ID |

### 跨資料庫差異

五個 DB `CREATE TABLE` 都含完整 16 欄。

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `Datetime` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `CanPrint` | `bit` | `NUMBER(1)` | `bit` | `DECIMAL(1,0)` | `DECIMAL(1,0)` |
| `Parameter` | `nvarchar(max)` | `clob` | `text` | `NVARCHAR(8000)` ⚠️ | `LVARCHAR(8000)` ⚠️ |

> ⚠️ DB2 / Informix `Parameter` 只有 8000 字元上限（同 FLOWTODO）。

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `CanPrint` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `ProjectID` | ✅ | ❌ | ❌ | ❌ | ❌ |

其他 4 DB 的 CREATE TABLE 都已含，但舊版升級要手動補。

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowNotifyHelper.cs` | 資料庫操作類別，提供 Select/Insert/Delete 方法 |
| `EEP.Flow/Activities/NotifyActivity.cs` | 知會活動定義，包含 SendTo 邏輯（Applicant/Manager/RefRole/RefUser/AllUsers） |
| `EEP.Flow/Instance.cs` | 流程實例類別，Notify() 方法觸發通知 |

---

## SendTo 類型（通知對象設定）

在 NotifyActivity 中，`SendTo` 屬性支援以下類型：

| 類型 | 說明 | 範例 |
|------|------|------|
| **Applicant** | 發起人 | `SendTo="Applicant"` |
| **Manager** | 目前角色的主管 | `SendTo="Manager"` |
| **ApplicantManager** | 發起人角色的主管 | `SendTo="ApplicantManager"` |
| **RefRole** | 參照欄位的角色值 | `SendTo="RefRole['RoleField']"` |
| **RefUser** | 參照欄位的使用者值 | `SendTo="RefUser['UserField']"` |
| **AllUsers** | 所有參與過流程的使用者 | `SendTo="AllUsers"` |

---

## 索引

```
INDEX ROLEINDEX ON FlowNotify (RoleID, Datetime)
INDEX USERINDEX ON FlowNotify (UserID, Datetime)
```

---

## 備註

- 與 FLOWTODO 結構相似，差異在於 FLOWNOTIFY 是知會（不需處理），FLOWTODO 是待辦（需處理）。
- 通知記錄建立後不會自動清除，會永久保留作為歷史記錄。
- `AllowEmpty` 屬性可設定是否允許通知對象為空白（預設 false，空白時會拋錯）。
- `LogHistory` 屬性可設定是否同時寫入流程歷史記錄（FLOWHISTORY）。
- `CanPrint` 屬性控制該通知是否支援列印功能。
