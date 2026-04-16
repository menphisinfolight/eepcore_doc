# USERS_LOG

## 用途

**使用者登入歷程記錄表**（User Login History Log）。

USERS_LOG 記錄每位使用者的登入與登出時間，用於追蹤登入歷史、統計使用時長。目前在程式碼中僅被讀取（統計 TOTALMINUTES），**寫入機制未在 C# 原始碼中找到**，可能由外部排程、資料庫觸發器或前端機制處理。

### 使用場景

| 場景 | 說明 |
|------|------|
| **使用時長統計** | `SecurityProvider.ExportUser()` 以 `SUM(DATEDIFF(mi, LOGINTIME, LOGOUTTIME))` 計算每位使用者的總使用分鐘數（TOTALMINUTES） |
| **使用者匯出** | 匯出使用者資料時，TOTALMINUTES 作為額外計算欄位附加於 USERS 資料中 |

### 關聯表

```
USERS_LOG ──(USERID)──> USERS  （使用者帳號）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUser()`：子查詢統計 TOTALMINUTES（SQL Server 用 `DATEDIFF(mi)`，Oracle 用 `ROUND(TO_NUMBER(LOGINTIME-LOGOUTTIME)*24*60)`） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增識別碼 |
| **USERID** | `varchar(20)` | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **LOGINTIME** | `datetime` | NOT NULL | 登入時間 |
| **LOGOUTTIME** | `datetime` | NOT NULL | 登出時間 |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle |
|------|-----------|-------|--------|
| ID | `int IDENTITY(1,1)` | `int AUTO_INCREMENT` | `integer` + createTablesIdentity 序列 |
| LOGINTIME | `datetime` | `datetime` | `date` |
| LOGOUTTIME | `datetime` | `datetime` | `date` |

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 備註

- 此表無對應的 infocommand 定義，也無 UpdateComponent。
- C# 原始碼中**未找到 INSERT/UPDATE 語句**，僅有 SELECT（統計用途）。登入/登出記錄的寫入機制不在 EEPCore SP7 的 C# 程式碼中，可能由其他機制處理。
- 與 USERS_LOGON 用途完全不同：USERS_LOG 是歷史記錄（登入+登出時間對），USERS_LOGON 是登入驗證次數控制。
- ExportUser() 針對不同資料庫使用不同的時間差計算方式：SQL Server 用 `DATEDIFF(mi, LOGINTIME, LOGOUTTIME)`，Oracle 用 `ROUND(TO_NUMBER(LOGINTIME - LOGOUTTIME) * 24 * 60)`。
