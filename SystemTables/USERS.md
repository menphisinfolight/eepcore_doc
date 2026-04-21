# USERS

## 用途

**使用者主檔**（Users Master）。

USERS 儲存 EEP Core 系統的所有使用者帳號資訊，包括帳密、個人資料、登入狀態、代理設定等。是權限模型「使用者 → 群組 → 選單權限」的起點。

### 使用場景

| 場景 | 說明 |
|------|------|
| **登入驗證** | `AccountProvider.LogonUser()` 查詢 USERID + PWD 驗證身份 |
| **密碼管理** | 變更密碼、重設密碼、密碼過期檢查（PWDDATE） |
| **帳號到期** | EXPIRYDATE 控制帳號有效期限 |
| **自動登入** | AUTOLOGIN 欄位控制是否啟用自動登入機制 |
| **MSAD 整合** | MSAD 欄位設定 Active Directory 帳號對應 |
| **LINE 綁定** | LINE 欄位儲存 LINE UserID，用於通知推播 |
| **代理機制** | AGENT / AGENTREADONLY 設定代理人 |
| **登入記錄** | `SetClientInfo()` 更新 LASTDATE / LASTTIME |
| **Azure AD** | DESCRIPTION 欄位存放 Azure AD 驗證碼（格式：`azureCode:xxxx,`） |
| **語系偏好** | LOCALE 欄位設定使用者偏好語言 |
| **使用者匯出** | `SecurityProvider.ExportUser()` 匯出完整使用者資料含群組/角色/使用時長 |

### 關聯表

```
USERS ──(USERGROUPS)──> GROUPS     （群組/角色歸屬）
      ──(USERMENUS)──> MENUTABLE  （直接選單權限）
      ──(USERCARDS)──> SYS_CARDS  （個人卡片權限）
      ──(SYS_USERS_AGENT)──> USERS（使用者代理）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | 核心：登入驗證、密碼管理、SetClientInfo、MSAD/Azure/LINE 整合、帳號註冊 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUser()`：匯出使用者含 GROUPNAMES/ROLENAMES/TOTALMINUTES |
| `SystemTable.Core/UserModule.cs` | `ucUser_onBeforeImport()`：匯入時加密密碼；匯出/匯入使用者資料 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `user` infocommand：`SELECT * FROM USERS`；`currentUser`：secField=USERID 過濾當前使用者 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` PK | NOT NULL | 使用者帳號識別碼 |
| **USERNAME** | `nvarchar(30)` | NULL | 使用者顯示名稱 |
| **PWD** | `nvarchar(10)` | NULL | 密碼（加密儲存） |
| **AGENT** | `nvarchar(20)` | NULL | 代理人 USERID |
| **AGENTREADONLY** | `nvarchar(1)` | NULL | 代理人是否唯讀（`Y`/`N`，後期擴充） |
| **CREATEDATE** | `nvarchar(8)` | NULL | 建立日期（格式：yyyyMMdd） |
| **CREATER** | `nvarchar(20)` | NULL | 建立者 USERID |
| **DESCRIPTION** | `nvarchar(100)` | NULL | 描述（也用於存放 Azure AD 驗證碼） |
| **EMAIL** | `nvarchar(40)` | NULL | 電子郵件 |
| **LASTTIME** | `nvarchar(8)` | NULL | 最後登入時間（格式：HHmmss） |
| **LASTDATE** | `nvarchar(8)` | NULL | 最後登入日期（格式：yyyyMMdd） |
| **AUTOLOGIN** | `nvarchar(1)` | NULL | 自動登入模式（`S`=超級自動、`X`=已啟用、空=停用） |
| **SIGNATURE** | `nvarchar(30)` | NULL | 電子簽章 |
| **MSAD** | `nvarchar(50)` | NULL | Active Directory 帳號對應 |
| **EXPIRYDATE** | `datetime` | NULL | 帳號到期日（後期擴充） |
| **LINE** | `nvarchar(50)` | NULL | LINE UserID（後期擴充） |
| **CALENDAR** | `nvarchar(200)` | NULL | 行事曆設定（後期擴充） |
| **PWDDATE** | `datetime` | NULL | 密碼最後變更日期（後期擴充） |
| **LOCALE** | `nvarchar(50)` | NULL | 使用者偏好語系（後期擴充） |
| **MOBILE** | `nvarchar(40)` | NULL | 手機號碼（後期擴充） |

### 跨資料庫差異

#### 欄位存在度（CREATE TABLE 新裝）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| USERID / USERNAME / PWD / AGENT / AGENTREADONLY | ✅ | ✅ | ✅ | ✅ | ✅ |
| CREATEDATE / CREATER / DESCRIPTION / EMAIL | ✅ | ✅ | ✅ | ✅ | ✅ |
| LASTTIME / LASTDATE / AUTOLOGIN / SIGNATURE | ✅ | ✅ | ✅ | ✅ | ✅ |
| EXPIRYDATE / LINE / CALENDAR | ✅ | ✅ | ✅ | ✅ | ✅ |
| **MSAD** | ✅ (50) | ✅ (50) | ✅ (50) | ⚠️ **(1)** | ⚠️ **(1)** |
| **MOBILE** | ✅ | ✅ | ✅ | ✅ | ❌ |
| **PWDDATE** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **LOCALE** | ✅ | ✅ | ✅ | ❌ | ❌ |

> ⚠️ **DB2 / Informix 的 `MSAD` 欄位建表腳本長度只有 `NVARCHAR(1)`**（其他 DB 是 50）。MSAD 是 Active Directory 帳號對應欄位，1 字元塞不下任何真實的 AD 帳號 — 看起來像腳本打字錯誤（`NVARCHAR(50)` 漏打成 `NVARCHAR(1)`）。**DB2 / Informix 環境若要用 LDAP/AD 整合，必須手動 `ALTER` 拉長此欄位**。

#### 主要型別對照

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `USERID` | `varchar(20)` | `varchar2(20)` | `varchar(20)` | `NVARCHAR(20)` | `NVARCHAR(20)` |
| `EXPIRYDATE` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `PWDDATE` | `datetime` | `date` | `datetime` | — | — |
| 其他 char 欄位 | `nvarchar(n)` | `varchar2(n)` | `nvarchar(n)` | `NVARCHAR(n)` | `NVARCHAR(n)` |

#### SP7 升級補欄位支援矩陣（舊版升級時是否自動 ALTER ADD）

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `AGENTREADONLY` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `EXPIRYDATE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `LINE` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `CALENDAR` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `PWDDATE` | ✅ | ❌ | ❌ | — | — |
| `LOCALE` | ✅ | ❌ | ❌ | — | — |
| `MOBILE` | ✅ | ❌ | ❌ | ❌ | — |

> **結論**：
> - **MSSQL**：7 個 SP7 新欄位全自動補
> - **Oracle**：只自動補 `EXPIRYDATE`，其餘 6 個欄位 CREATE TABLE 雖已含，但**舊表升級時不會自動補**
> - **MySQL / DB2 / Informix**：建表腳本**完全沒有 `ALTER TABLE ADD`**，而且 DB2 / Informix 連 CREATE TABLE 本身就缺 `PWDDATE` / `LOCALE`（+ Informix 缺 `MOBILE`）
> - MSSQL 的 `PWDDATE` ALTER 是 `nvarchar(200)`，但 CREATE TABLE 是 `datetime` — 程式實際讀寫仍用日期格式，這是很古早的腳本殘留，正常不會觸發（CREATE TABLE 已經有 datetime 欄位）

#### 手動補欄位 / 修欄位 SQL 範本

```sql
-- DB2：補 PWDDATE、LOCALE 並修正 MSAD 長度
ALTER TABLE USERS ADD COLUMN PWDDATE TIMESTAMP;
ALTER TABLE USERS ADD COLUMN LOCALE NVARCHAR(50);
ALTER TABLE USERS ALTER COLUMN MSAD SET DATA TYPE NVARCHAR(50);

-- Informix：補 PWDDATE、LOCALE、MOBILE 並修正 MSAD 長度
ALTER TABLE "USERS" ADD ("PWDDATE" DATETIME YEAR TO SECOND);
ALTER TABLE "USERS" ADD ("LOCALE" NVARCHAR(50));
ALTER TABLE "USERS" ADD ("MOBILE" NVARCHAR(40));
ALTER TABLE "USERS" MODIFY ("MSAD" NVARCHAR(50));

-- MySQL：舊版升級時所有 SP7 新欄位都要自己補
ALTER TABLE USERS ADD AGENTREADONLY nvarchar(1) NULL;
ALTER TABLE USERS ADD EXPIRYDATE datetime NULL;
ALTER TABLE USERS ADD LINE nvarchar(50) NULL;
ALTER TABLE USERS ADD CALENDAR nvarchar(200) NULL;
ALTER TABLE USERS ADD PWDDATE datetime NULL;
ALTER TABLE USERS ADD LOCALE nvarchar(50) NULL;
ALTER TABLE USERS ADD MOBILE nvarchar(40) NULL;

-- Oracle：除 EXPIRYDATE 外其餘 6 個升級都要手動補
ALTER TABLE USERS ADD AGENTREADONLY varchar2(1) NULL;
ALTER TABLE USERS ADD LINE varchar2(50) NULL;
ALTER TABLE USERS ADD CALENDAR varchar2(200) NULL;
ALTER TABLE USERS ADD PWDDATE date NULL;
ALTER TABLE USERS ADD LOCALE varchar2(50) NULL;
ALTER TABLE USERS ADD MOBILE varchar2(40) NULL;
```

---

## 主鍵

```
PRIMARY KEY (USERID)
```

---

## 預設資料

```sql
INSERT INTO USERS(USERID, USERNAME, PWD, MSAD, AUTOLOGIN) VALUES('001', 'TEST', '', 'N', 'S');
```

系統預設建立測試帳號 `001`，AUTOLOGIN='S'（超級自動登入，不需密碼）。

---

## 資料生命週期

```
登入：AccountProvider.LogonUser()
      → SELECT * FROM USERS WHERE USERID = @user
      → 驗證 PWD → SetClientInfo() → UPDATE LASTDATE + LASTTIME

登入後設定：AccountProvider.SetClientInfo()
      → UPDATE USERS SET LASTDATE=@date, LASTTIME=@time WHERE USERID=@user
      → 查詢 USERGROUPS → 設定 ClientInfo.Groups/Roles/OrgRoles

匯入：UserModule.ucUser_onBeforeImport()
      → 加密 PWD：EncryptHelper.EncryptPassword(user, pwd)
```

---

## 備註

- PWD 欄位長度 `nvarchar(10)`，密碼以 EncryptHelper 加密後儲存。
- AUTOLOGIN='S' 為超級使用者模式，登入時不需密碼驗證。
- DESCRIPTION 具有雙重用途：一般描述 + Azure AD 驗證碼暫存。
- 多個欄位為後期擴充（SQL 腳本中有 ALTER TABLE）：AGENTREADONLY、EXPIRYDATE、LINE、CALENDAR、PWDDATE、LOCALE、MOBILE。
- LINE 綁定流程：使用者透過 LINE Bot 推播連結 → 輸入 EEP 帳密 → AccountProvider 寫入 LINE UserID。
