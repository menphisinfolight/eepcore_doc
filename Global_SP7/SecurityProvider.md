# SecurityProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/SecurityProvider.cs` |
| 行數 | 677 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` → `BaseProvider` |

## 用途

設計端的安全管理模組，負責使用者/群組/角色資料的 CRUD 操作、權限資料匯出（Excel）、AD 帳號匯入、線上使用者管理、日誌匯出，以及方案重新命名等功能。

## ProcessRequest 模式

| mode | 方法 | 說明 |
|------|------|------|
| `load` | `GetDataset(command, options)` | 查詢資料集（menu 為樹狀、org 為組織樹、其他為一般表格） |
| `save` | `UpdateDataset(command, datas, options)` | 儲存資料集 |
| `renameSolution` | `RenameSolution(oldName, newName)` | 重新命名方案（更新 MENUITEMTYPE / MENUTABLE / MENUFAVOR） |
| `importAccount` | `ImportAccount(domain, user, password, importMode)` | 從 LDAP/AD 匯入使用者與群組 |
| `getOnlineUsers` | `GetUsers()` | 取得目前線上使用者清單（SessionInfos） |
| `removeSession` | `RemoveUser(id)` | 強制移除指定使用者的 Session |
| `exportLog` | (委派 LogProvider) | 匯出系統日誌為 Excel |
| `exportSqlLog` | (委派 LogProvider) | 匯出 SQL 日誌為 Excel |
| `exportUser` | `ExportUser()` | 匯出所有使用者資料為 Excel |
| `exportUserAccess` | `ExportUserAccess(user)` | 匯出指定使用者的權限清單為 Excel |
| `exportGroup` | `ExportGroup()` | 匯出所有群組/角色資料為 Excel |
| `exportGroupUser` | `ExportGroupUser(group)` | 匯出指定群組的成員與權限為 Excel |
| `exportTreeNode` | `ExportTreeNode(menu)` | 匯出指定選單節點的存取權限為 Excel |
| `exportOrg` | `ExportOrg(menu)` | 匯出組織架構為 Excel |

## 關鍵方法

### 資料查詢與更新

| 方法 | 說明 |
|------|------|
| `GetDataset(command, options)` | 使用 `DatabaseType.Split`，依 command 類型回傳不同格式：`menu` → 樹狀（MENUID/CAPTION/PARENT）、`org` → 組織樹（ORG_NO/ORG_DESC/UPPER_ORG）、`SYS_CARDS` → 卡片樹、其他 → 一般表格 |
| `UpdateDataset(command, datas, options)` | 儲存資料，若 `config.validateXss == "false"` 則停用 XSS 驗證 |

### AD 帳號匯入

| 方法 | 說明 |
|------|------|
| `ImportAccount(domain, user, password, mode)` | 透過 `LDAPHelper` 取得 AD 使用者與群組，以 `ImportMode`（預設 merge）寫入 USERS / GROUPS / USERGROUPS |

### Excel 匯出系列

所有匯出方法的共通模式：
1. 透過 `DatabaseHelper` 執行 SQL 查詢
2. 組合 `columns`（欄位定義 JArray）
3. 呼叫 `FileProvider.ExportToExcel(datas, columns, "")` 產生 Excel

| 方法 | SQL 查詢重點 | 匯出欄位 |
|------|-------------|---------|
| `ExportUser()` | USERS + USERGROUPS 聯結取得群組/角色名稱 + 登入時數統計（USERS_LOG） | USERID, USERNAME, CREATEDATE, EMAIL, AUTOLOGIN, CREATER, LASTDATE, MSAD, EXPIRYDATE, PWDDATE, GROUPNAMES, ROLENAMES, TOTALMINUTES, DESCRIPTION |
| `ExportUserAccess(user)` | USERS + USERMENUS + MENUTABLE + GROUPS + GROUPMENUS 聯集 | ID, NAME, ISROLE, CAPTION, ALLOWADD, ALLOWUPDATE, ALLOWDELETE |
| `ExportGroup()` | GROUPS + USERGROUPS + USERS 聯結 | GROUPID, GROUPNAME, ISROLE, USERNAME |
| `ExportGroupUser(group)` | 與 ExportUserAccess 類似但以群組為條件 | ID, NAME, ISROLE, CAPTION, ALLOWADD, ALLOWUPDATE, ALLOWDELETE |
| `ExportTreeNode(menu)` | MENUTABLE + USERMENUS/GROUPMENUS 聯結 | MENUID, CAPTION, ID, NAME, ISROLE, ALLOWADD, ALLOWUPDATE, ALLOWDELETE |
| `ExportOrg(menu)` | 先以 CTE 計算 ORG_TREE（組織排序），再查詢 SYS_ORG + SYS_ORGROLES + GROUPS | 組織層級, 組織編號, 組織名稱, 主管名稱, 職級, 成員角色, 成員角色名稱 |

### 格式化輔助

| 方法 | 說明 |
|------|------|
| `FormatDateTime(date, time)` | 將 `"20240101"` + `"120000"` 格式化為 `"2024-01-01 12:00:00"` |
| `FormatType(localeItem, value)` | AUTOLOGIN 代碼轉多語系文字（S → 管理員、U → 使用者、X → 停用 等） |
| `FormatVersion(localeItem, value)` | CREATER 版本代碼轉文字（L → Lite、E → Enterprise、U → Ultimate、S → Unlimited） |

## 備註

- 所有資料操作使用 `DatabaseType.Split`（分離式資料庫連線）。
- Excel 匯出 SQL 同時支援 SQL Server 與 Oracle（使用 `STUFF/FOR XML PATH` vs `LISTAGG`）。
- `ExportOrg` 先執行 UPDATE SQL 計算 ORG_TREE 欄位（組織排序路徑），再執行查詢匯出。
- `RenameSolution` 目前只更新 DB 中的 MENUITEMTYPE/MENUTABLE/MENUFAVOR，檔案複製邏輯可能不完整（使用 `File.Copy(oldName, newName)` 而非目錄操作）。
- 線上使用者管理透過靜態的 `SessionInfos` 類別實現。
