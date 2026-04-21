# SP7 跨資料庫欄位差異盤點

> EEP Core 的系統資料表提供 MSSQL / Oracle / MySQL / DB2 / Informix 五套 DDL 腳本（`EEPWebClient.Core/wwwroot/sql/{sql,oracle,mysql,db2,informix}/systemTables.sql`）。本頁集中記錄**五套腳本彼此不一致的地方** —— 哪些欄位在某 DB 缺失、哪些型別或長度不一致、哪些新版 ALTER 補欄位邏輯只在 MSSQL 有。客戶升級 SP7 前務必按此表比對。

## TL;DR 風險等級

| ⚠️ 風險 | 受影響 DB | 影響功能 |
|---------|----------|---------|
| `USERS.MSAD = NVARCHAR(1)` | DB2 / Informix | LDAP/AD 整合 |
| `GROUPMENUS` / `USERMENUS` 缺 3 個 `ALLOW*` | DB2 / Informix | 選單細權限（新增 / 修改 / 刪除） |
| `FLOWHISTORY` 缺 `Duration` / `ExpDatetime` | Informix | SLA 統計、預期完成時間 |
| `SYS_XLSFILES` 缺 `HEADFORMAT` | Oracle / MySQL / DB2 / Informix | Excel 報表表頭 |
| `SYS_XLSFILES` 缺 `KEYFIELDS` / `NAMEFIELDS` | DB2 / Informix | Excel 報表主鍵 / 名稱欄位 |
| `SYS_DOCFILES` 缺 `SUBDETAILTABLE` | DB2 | 報表第三層明細 |
| `SYS_USERDEF` 欄位名 `DATE` vs `DEFDATE` | DB2 / Informix | 跨 DB 查詢 Date 欄位 |
| `SYS_USERDEF` 缺 `RFORMAT` | DB2 | 報表格式化 |
| `FLOWHISTORY.Duration = NUMBER(1)` | Oracle | Duration 只能存 0-9 |
| 升級 ALTER 覆蓋不全 | 全部（MSSQL 以外）| 舊版升級後新欄位缺 |

## ALTER TABLE 數量總覽

升級腳本自動補欄位 / 修欄位的 `ALTER TABLE` 語句總數：

| DB | 數量 | 意義 |
|----|:-:|------|
| MSSQL | 70 | 完整升級自動化 |
| Oracle | 11 | 僅少數欄位 |
| MySQL | 0 | 無升級邏輯 |
| DB2 | 0 | 用 `CREATE OR REPLACE`，客戶若保留舊表則新欄位全缺 |
| Informix | 0 | 用 `CREATE IF NOT EXISTS`，舊表不補 |

## 逐表欄位存在度（只列有差異的欄位）

### COLDEF（欄位定義中繼資料）

新裝：五個 DB CREATE TABLE 都完整 36 欄。差異在**升級 ALTER**：

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `CANEDIT` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_MERGE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_VISIBLE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_TOTAL` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_READONLY` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `SHOWAPP` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `DESCRIPTION` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `GROUPTITLE` | ✅ | ❌ | ❌ | ❌ | ❌ |
| PK 長度拉到 200 | ✅ | ❌ | ❌ | ❌ | ❌ |

PK 欄位長度不一致：DB2 `TABLE_NAME` / `FIELD_NAME` 只有 **20 字元**、Informix 50，其他 DB 200。

### USERS（使用者主檔）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| **MSAD** 長度 | 50 | 50 | 50 | ⚠️ **1** | ⚠️ **1** |
| `MOBILE` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `PWDDATE` | ✅ | ✅ | ✅ | ❌ | ❌ |
| `LOCALE` | ✅ | ✅ | ✅ | ❌ | ❌ |
| 升級 ALTER `AGENTREADONLY` | ✅ | ❌ | ❌ | ❌ | ❌ |
| 升級 ALTER `EXPIRYDATE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| 升級 ALTER `LINE` / `CALENDAR` / `PWDDATE` / `LOCALE` / `MOBILE` | ✅ | ❌ | ❌ | ❌ | ❌ |

**DB2 / Informix 的 `MSAD = NVARCHAR(1)` 幾乎確定是 DDL 腳本 typo**（`NVARCHAR(50)` 漏打 5）。

### GROUPMENUS / USERMENUS（選單權限）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `ALLOWADD` | ✅ | ✅ | ✅ | ❌ | ❌ |
| `ALLOWUPDATE` | ✅ | ✅ | ✅ | ❌ | ❌ |
| `ALLOWDELETE` | ✅ | ✅ | ✅ | ❌ | ❌ |
| 升級 ALTER | ✅ | ❌ | ❌ | ❌ | ❌ |

DB2 / Informix **新裝 CREATE TABLE 就完全沒有這 3 個欄位**（不是升級遺漏、是腳本壓根沒寫）。

`GROUPMENUS.GROUPID`：MSSQL / Informix 50、Oracle / MySQL / DB2 只 20 字元。

### USERGROUPS（使用者-群組）

PK 欄位長度跨 DB 相反：

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `USERID` 長度 | 20 | 20 | 20 | 20 | ⚠️ **50** |
| `GROUPID` 長度 | 50 | 50 | 50 | ⚠️ **20** | 20 |
| 升級 ALTER `LINE` | ✅ | ❌ | ❌ | ❌ | ❌ |

### MENUCHECKLOG（版本管理歷史）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `USERID`（操作者） | ✅ | ❌ | ❌ | ❌ | ❌ |
| 升級 ALTER `USERID` | ✅ | — | — | — | — |

`FILECONTENT` 型別差異：MSSQL `image`、Oracle `CLOB`、MySQL `LONGBLOB`、DB2 `BLOB(10M)`、Informix `BLOB`。

### FLOWINSTANCE / FLOWTODO / FLOWNOTIFY / FLOWHISTORY / FLOWCOMMENT / FLOWCOMMENTDETAIL

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| FLOWTODO `CanEdit` / `CanReject` / `CanPrint` / `ExpDatetime` / `ProjectID` 升級 ALTER | ✅ | ❌ | ❌ | ❌ | ❌ |
| FLOWNOTIFY `CanPrint` / `ProjectID` 升級 ALTER | ✅ | ❌ | ❌ | ❌ | ❌ |
| FLOWHISTORY `ProjectID` 升級 ALTER | ✅ | ❌ | ❌ | ❌ | ❌ |
| **FLOWHISTORY `Duration`** | ✅ | ⚠️ `NUMBER(1)` | ✅ | ✅ | ❌ |
| **FLOWHISTORY `ExpDatetime`** | ✅ | ✅ | ✅ | ✅ | ❌ |
| FLOWHISTORY `Duration` / `ExpDatetime` 升級 ALTER | ✅ | ✅（覆蓋 CREATE 的 NUMBER(1) 為 NUMBER(10,2)）| ❌ | ❌ | ❌ |
| FLOWCOMMENT / FLOWCOMMENTDETAIL `ProjectID` 升級 ALTER | ✅ | ❌ | ❌ | ❌ | ❌ |

**Oracle FLOWHISTORY.Duration 陷阱**：CREATE TABLE 寫 `NUMBER(1)`（只能存 0-9），升級 ALTER 才修正為 `NUMBER(10,2)` —— 所以**新裝 Oracle 的舊腳本若只跑 CREATE 沒跑 ALTER，Duration 會爛**。

**Informix FLOWHISTORY 新裝缺 `Duration` 與 `ExpDatetime`**，SLA 相關功能必爆。

流程表共通型別差異：
- `Datetime` / `ExpDatetime`：MSSQL `datetime` / Oracle `date` / DB2 `TIMESTAMP` / Informix `DATETIME YEAR TO SECOND`
- `Parameter`：MSSQL `nvarchar(max)` / Oracle `clob` / MySQL `text` / DB2 `NVARCHAR(8000)` / Informix `LVARCHAR(8000)` —— **DB2 / Informix 8000 字元上限**
- Boolean (`Can*`, `IsRead`, `Disabled`)：MSSQL `bit` / Oracle `NUMBER(1)` or `CHAR(1)` / MySQL `bit` / DB2 `DECIMAL(1,0)` / Informix `DECIMAL(1,0)` —— 讀取端需處理多種表示

### SYS_MESSENGER

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `PARAS` 長度 | `nvarchar(max)` | 4000 | `text` | 4096 | ⚠️ **1024** |
| 升級 ALTER `ID` | ✅ | ❌ | ❌ | ❌ | ❌ |

### SYS_DOCFILES（Word 報表定義）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| **`SUBDETAILTABLE`** | ✅ | ✅ | ✅ | ❌ | ✅ |
| `REPORTFORMAT` 長度 | `nvarchar(max)` | `clob` | `text` | 8000 | 8000 |
| 升級 ALTER `DETAIL2TABLE` / `DETAIL3TABLE` / `SUBDETAILTABLE` | ✅ | ❌ | ❌ | ❌ | ❌ |

### SYS_XLSFILES（Excel 報表定義）

**這張表最破**。

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| **`KEYFIELDS`** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **`NAMEFIELDS`** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **`HEADFORMAT`** | ✅ | ❌ | ❌ | ❌ | ❌ |
| `QUERYFORMAT` 長度 | `nvarchar(max)` | `clob` | `text` | 2000 | 2000 |
| `REPORTFORMAT` 長度 | `nvarchar(max)` | `clob` | `text` | 14000 | 14000 |
| 升級 ALTER（5 個欄位）| ✅ | ❌ | ❌ | ❌ | ❌ |

除 MSSQL 外全部 DB 新裝都缺 `HEADFORMAT`，DB2 / Informix 再缺 `KEYFIELDS` / `NAMEFIELDS`。

### SYS_TRSFILES（轉換定義）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `TRSFORMAT` 長度 | `nvarchar(max)` | `clob` | `text` | 8000 | 8000 |
| 升級 ALTER `DETAIL2TABLE` / `DETAIL3TABLE` | ✅ | ❌ | ❌ | ❌ | ❌ |

### SYS_CARDS

**五個 DB CREATE TABLE 完整一致**（含 SP7 的 `COLOR` / `SEQ_NO` / `SIZE` / `ITEMTYPE`）。只有 MSSQL 有升級 ALTER，但其他 DB 的 CREATE 本就完整，新裝沒問題。

### SYS_SCHEDULE

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `InvokeTime` 長度 | `nvarchar(max)` | `clob` | `text` | 2000 | 2000 |
| 升級 ALTER `Disabled` | ✅ | ❌ | ❌ | ❌ | ❌ |

### SYS_USERDEF（使用者報表定義）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| **日期欄位名** | `DEFDATE` | `DEFDATE` | `DEFDATE` | ⚠️ `` `DATE` `` | ⚠️ `DATE` |
| **`RFORMAT`** | ✅ | ✅ | ✅ | ❌ | ✅ |
| 升級 ALTER `GROUPCOLUMN` / `RFORMAT` | ✅ | ✅ | ❌ | ❌ | ❌ |

DB2 還用反引號 `` ` `` 包欄位名（MySQL 語法，DB2 不認），像混用了不同 DB 的腳本樣板。

## 手動補欄位 / 修欄位 SQL 範本（優先順序）

### 1. 修正型別錯誤

```sql
-- DB2 / Informix：USERS.MSAD 拉長
ALTER TABLE USERS ALTER COLUMN MSAD SET DATA TYPE NVARCHAR(50);            -- DB2
ALTER TABLE "USERS" MODIFY ("MSAD" NVARCHAR(50));                           -- Informix

-- Oracle：FLOWHISTORY.Duration 改成合理長度
ALTER TABLE "FLOWHISTORY" MODIFY ("Duration" NUMBER(10,2));

-- DB2 / Informix：SYS_USERDEF 欄位名修正
ALTER TABLE SYS_USERDEF RENAME COLUMN "DATE" TO DEFDATE;                    -- DB2
ALTER TABLE "SYS_USERDEF" RENAME COLUMN "DATE" TO "DEFDATE";                -- Informix
```

### 2. DB2 / Informix 補缺欄位

```sql
-- GROUPMENUS / USERMENUS（DB2）
ALTER TABLE GROUPMENUS ADD COLUMN ALLOWADD CHAR(1);
ALTER TABLE GROUPMENUS ADD COLUMN ALLOWUPDATE CHAR(1);
ALTER TABLE GROUPMENUS ADD COLUMN ALLOWDELETE CHAR(1);
ALTER TABLE USERMENUS ADD COLUMN ALLOWADD CHAR(1);
ALTER TABLE USERMENUS ADD COLUMN ALLOWUPDATE CHAR(1);
ALTER TABLE USERMENUS ADD COLUMN ALLOWDELETE CHAR(1);

-- GROUPMENUS / USERMENUS（Informix）
ALTER TABLE "GROUPMENUS" ADD ("ALLOWADD" CHAR(1));
ALTER TABLE "GROUPMENUS" ADD ("ALLOWUPDATE" CHAR(1));
ALTER TABLE "GROUPMENUS" ADD ("ALLOWDELETE" CHAR(1));
ALTER TABLE "USERMENUS" ADD ("ALLOWADD" CHAR(1));
ALTER TABLE "USERMENUS" ADD ("ALLOWUPDATE" CHAR(1));
ALTER TABLE "USERMENUS" ADD ("ALLOWDELETE" CHAR(1));

-- USERS（DB2 補 PWDDATE/LOCALE、Informix 再補 MOBILE）
ALTER TABLE USERS ADD COLUMN PWDDATE TIMESTAMP;
ALTER TABLE USERS ADD COLUMN LOCALE NVARCHAR(50);
ALTER TABLE "USERS" ADD ("PWDDATE" DATETIME YEAR TO SECOND);
ALTER TABLE "USERS" ADD ("LOCALE" NVARCHAR(50));
ALTER TABLE "USERS" ADD ("MOBILE" NVARCHAR(40));

-- FLOWHISTORY（Informix）
ALTER TABLE "FlowHistory" ADD ("Duration" DECIMAL(8,2));
ALTER TABLE "FlowHistory" ADD ("ExpDatetime" DATETIME YEAR TO SECOND);

-- SYS_DOCFILES（DB2）
ALTER TABLE SYS_DOCFILES ADD COLUMN SUBDETAILTABLE NVARCHAR(50);

-- SYS_XLSFILES（DB2 / Informix 缺 3 欄位）
ALTER TABLE SYS_XLSFILES ADD COLUMN KEYFIELDS NVARCHAR(100);
ALTER TABLE SYS_XLSFILES ADD COLUMN NAMEFIELDS NVARCHAR(100);
ALTER TABLE SYS_XLSFILES ADD COLUMN HEADFORMAT NVARCHAR(14000);
ALTER TABLE "SYS_XLSFILES" ADD ("KEYFIELDS" NVARCHAR(100));
ALTER TABLE "SYS_XLSFILES" ADD ("NAMEFIELDS" NVARCHAR(100));
ALTER TABLE "SYS_XLSFILES" ADD ("HEADFORMAT" LVARCHAR(14000));

-- SYS_XLSFILES（Oracle / MySQL 補 HEADFORMAT）
ALTER TABLE SYS_XLSFILES ADD HEADFORMAT clob NULL;                          -- Oracle
ALTER TABLE SYS_XLSFILES ADD HEADFORMAT text NULL;                          -- MySQL

-- SYS_USERDEF.RFORMAT（DB2）
ALTER TABLE SYS_USERDEF ADD COLUMN RFORMAT NVARCHAR(4000);
```

### 3. 舊版升級自動 ALTER 缺漏的欄位

除 MSSQL 外所有 DB 都存在此問題。建議做法：**客戶現場跑完建表後，DBA 執行「補欄位檢查腳本」**，逐欄位 `SELECT` 驗證是否存在、不存在再 ALTER ADD。各欄位清單見前面各表章節。

## 驗證腳本

用來快速檢查當下 DB 是否有缺欄位（範例：MSSQL，其他 DB 依語法調整）：

```sql
-- MSSQL 版
SELECT OBJECT_NAME(c.object_id) AS table_name, c.name AS column_name
FROM sys.columns c
WHERE OBJECT_NAME(c.object_id) IN (
    'COLDEF','USERS','GROUPMENUS','USERMENUS','USERGROUPS','MENUCHECKLOG',
    'SYS_MESSENGER','SYS_DOCFILES','SYS_XLSFILES','SYS_TRSFILES',
    'FlowInstance','FlowToDo','FlowNotify','FlowHistory','FlowComment','FlowCommentDetail',
    'SYS_CARDS','SYS_SCHEDULE','SYS_USERDEF'
)
ORDER BY table_name, c.column_id;

-- Oracle 版
SELECT TABLE_NAME, COLUMN_NAME FROM USER_TAB_COLUMNS
WHERE TABLE_NAME IN ('COLDEF','USERS',...)
ORDER BY TABLE_NAME, COLUMN_ID;

-- MySQL 版
SELECT TABLE_NAME, COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = DATABASE()
  AND TABLE_NAME IN ('COLDEF','USERS',...)
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

## 盤點方法 / 可驗證性

本文的結論皆來自對以下 5 個 DDL 檔的逐欄位 diff：

- `wwwroot/sql/sql/systemTables.sql`（MSSQL，70 條 ALTER）
- `wwwroot/sql/oracle/systemTables.sql`（Oracle，11 條 ALTER）
- `wwwroot/sql/mysql/systemTables.sql`（MySQL，0 條 ALTER）
- `wwwroot/sql/db2/systemTables.sql`（DB2，0 條 ALTER，用 `CREATE OR REPLACE`）
- `wwwroot/sql/informix/systemTables.sql`（Informix，0 條 ALTER，用 `CREATE IF NOT EXISTS`）

若 SP7 更新到新版，請重跑 diff 並更新此文件。