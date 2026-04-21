# USERGROUPS

## 用途

**使用者-群組關聯表**（User-Group Membership）。

USERGROUPS 定義使用者與群組/角色的多對多歸屬關係。一個使用者可屬於多個群組，一個群組可包含多個使用者。透過此關聯，使用者繼承群組的選單權限（GROUPMENUS）、卡片權限（GROUPCARDS），並在流程引擎中作為角色判斷依據。

### 使用場景

| 場景 | 說明 |
|------|------|
| **登入時載入群組** | `AccountProvider.SetClientInfo()` 查詢 `USERGROUPS LEFT JOIN GROUPS`，將 Groups 和 Roles 寫入 ClientInfo |
| **權限繼承** | `runtimeMenu_onBeforeExecuteSQL()` 以 ClientInfo.Groups 查詢 GROUPMENUS 取得群組繼承的選單 |
| **群組→使用者管理** | `userGroups` infocommand：以群組為主查看成員（LEFT JOIN USERS 取 USERNAME） |
| **使用者→群組管理** | `groupUsers` infocommand：以使用者為主查看所屬群組（LEFT JOIN GROUPS 取 GROUPNAME） |
| **未加入使用者查詢** | `userGroup` infocommand：LEFT JOIN + WHERE GROUPID IS NULL 篩出未加入指定群組的使用者 |
| **未加入群組查詢** | `groupUser` infocommand：LEFT JOIN + WHERE USERID IS NULL 篩出使用者未加入的群組 |
| **組織圖主管** | `org` infocommand：子查詢 USERGROUPS 取得 ORG_MAN 對應的 USERNAME |
| **當前使用者群組** | `currentGroup` infocommand：以 secField=USERID 過濾，顯示當前使用者的群組清單 |
| **使用者匯出** | `SecurityProvider.ExportUser()` 以 USERGROUPS JOIN GROUPS 組合 GROUPNAMES / ROLENAMES |
| **使用者匯入** | `UserModule` 匯入時 DELETE + INSERT 重建 USERGROUPS |

### 關聯表

```
USERS ──(USERGROUPS)──> GROUPS
                     └─> GROUPMENUS → MENUTABLE
                     └─> GROUPCARDS → SYS_CARDS
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `SetClientInfo()`：登入後查詢 USERGROUPS 載入 Groups/Roles 到 ClientInfo |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUserAccess()`：USERGROUPS 取 GROUPID 查 GROUPMENUS；`ExportUser()`：組合 GROUPNAMES/ROLENAMES |
| `SystemTable.Core/UserModule.cs` | `ucUserGroup_onBeforeApply/Insert()`、`ucGroupUser_onBeforeApply/Insert()`：群組成員管理 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `userGroups`/`groupUsers`/`userGroup`/`groupUser`/`currentGroup`：5 個 infocommand |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **GROUPID** | `varchar(50)` PK | NOT NULL | 群組/角色識別碼（對應 GROUPS.GROUPID） |
| **LINE** | `nvarchar(50)` | NULL | 群組專屬的 LINE 通知 ID（後期擴充） |

### 跨資料庫差異

#### 欄位存在度

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `USERID` / `GROUPID` | ✅ | ✅ | ✅ | ✅ | ✅ |
| **`LINE`** | ✅ | ✅ | ✅ | ✅ | ✅ |

CREATE TABLE 五個 DB 都有完整欄位，**無差異**。

#### 型別對照（PK 欄位長度嚴重不一致）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `USERID` | `varchar(20)` | `varchar2(20)` | `varchar(20)` | `NVARCHAR(20)` | ⚠️ `NVARCHAR(50)` |
| `GROUPID` | `varchar(50)` | `varchar2(50)` | `varchar(50)` | ⚠️ `NVARCHAR(20)` | `NVARCHAR(20)` |
| `LINE` | `nvarchar(50)` | `varchar2(50)` | `nvarchar(50)` | `NVARCHAR(50)` | `NVARCHAR(50)` |

> ⚠️ **PK 長度 DB 間不一致**：DB2 的 `GROUPID` 只有 20（其他 50），Informix 的 `USERID` 為 50（其他 20）。若群組碼常超過 20，DB2 會出包；使用者帳號較長時，MSSQL/Oracle/MySQL/DB2 的 20 字元也可能不夠。

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `LINE` | ✅ | ❌ | ❌ | ❌ | ❌ |

只有 MSSQL 有升級補 `LINE` 欄位邏輯；其他 4 個 DB 的 CREATE TABLE 都已含 `LINE`，但舊版升級的 MySQL / DB2 / Informix / Oracle 需要手動補：

```sql
-- Oracle
ALTER TABLE USERGROUPS ADD LINE varchar2(50) NULL;
-- MySQL
ALTER TABLE USERGROUPS ADD LINE nvarchar(50) NULL;
-- DB2
ALTER TABLE USERGROUPS ADD COLUMN LINE NVARCHAR(50);
-- Informix
ALTER TABLE "USERGROUPS" ADD ("LINE" NVARCHAR(50));
```

---

## 主鍵

```
PRIMARY KEY (USERID, GROUPID)
```

---

## 資料生命週期

```
加入群組（群組→使用者視角）：Board.cshtml → ucUserGroup
      → ucUserGroup_onBeforeApply()：DELETE
      → ucUserGroup_onBeforeInsert()：先 DELETE 同鍵 → INSERT

加入群組（使用者→群組視角）：Board.cshtml → ucGroupUser
      → ucGroupUser_onBeforeApply()：DELETE
      → ucGroupUser_onBeforeInsert()：先 DELETE 同鍵 → INSERT

登入載入：AccountProvider.SetClientInfo()
      → SELECT * FROM USERGROUPS LEFT JOIN GROUPS
        ON USERGROUPS.GROUPID = GROUPS.GROUPID
        WHERE USERID = @user
      → clientInfo.Groups = 所有 GROUPID
      → clientInfo.Roles = ISROLE='Y' 的 GROUPID

使用者匯入：UserModule
      → DELETE FROM USERGROUPS WHERE GROUPID IN (...)
      → INSERT INTO USERGROUPS (USERID, GROUPID) VALUES (...)
```

---

## 備註

- EveryOne 群組（GROUPID='00'）不需要在 USERGROUPS 中明確加入。安全查詢和選單載入時會自動在 Groups 列表中加入 '00'。
- AccountProvider 區分 Groups（所有群組）和 Roles（ISROLE='Y' 的群組），兩者都從 USERGROUPS 查詢。
- Oracle 版本的 SetClientInfo 查詢使用字串拼接（`WHERE USERID = '{user}'`），非參數化查詢。
- LINE 欄位為後期擴充（SQL 腳本中有 ALTER TABLE 補欄位邏輯），用於群組級別的 LINE 通知。
- GROUPID 欄位長度從 20 擴展到 50（SQL 腳本中有 ALTER TABLE ALTER COLUMN）。
