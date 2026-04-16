# USERDEVICES

## 用途

**使用者裝置註冊表**（User Device Registration）。

USERDEVICES 儲存使用者的行動裝置註冊資訊，用於推播通知（APNs / FCM）。每個使用者可註冊多個裝置，支援裝置啟用/停用、Token 管理、到期日控制。

> **MAUI APP 模組專用**：此表由 EEP MAUI APP 模組操作，EEPCore Web 端的 C# 程式碼和前端 JavaScript 均無直接引用。

### 使用場景

| 場景 | 說明 |
|------|------|
| **行動裝置推播** | 註冊使用者裝置的推播 Token（REGID），發送通知時查詢目標裝置 |
| **多裝置管理** | 以 USERID + UUID 為主鍵，單一使用者可註冊多個裝置 |
| **裝置啟停控制** | ACTIVE 欄位控制裝置是否接收推播 |
| **Token 管理** | REGID 儲存推播 Token，TOKENID 儲存認證 Token |

### 關聯表

```
USERDEVICES ──(USERID)──> USERS  （使用者帳號）
```

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `NVARCHAR(50)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **UUID** | `NVARCHAR(50)` PK | NOT NULL | 裝置唯一識別碼 |
| **ACTIVE** | `VARCHAR(1)` | NULL | 是否啟用（`Y`/`N`） |
| **CREATEDATE** | `DATETIME` | NULL | 裝置註冊日期 |
| **LOGINDATE** | `DATETIME` | NULL | 最後登入日期 |
| **EXPIRYDATE** | `DATETIME` | NULL | 裝置到期日 |
| **REGID** | `NVARCHAR(1024)` | NULL | 推播註冊 ID（APNs Device Token / FCM Registration Token） |
| **TOKENID** | `NVARCHAR(1024)` | NULL | 認證 Token ID |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| CREATEDATE | `DATETIME` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| LOGINDATE | `DATETIME` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| EXPIRYDATE | `DATETIME` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| REGID | `NVARCHAR(1024)` | `varchar2(1024)` | `nvarchar(1024)` | `NVARCHAR(1024)` |
| TOKENID | `NVARCHAR(1024)` | `varchar2(1024)` | `nvarchar(1024)` | `NVARCHAR(1024)` |

---

## 主鍵

```
PRIMARY KEY (USERID, UUID)
```

---

## SQL 建表特殊邏輯（SQL Server）

SQL Server 的建表腳本包含大小寫敏感性修正邏輯：

```sql
IF OBJECT_ID('USERDEVICES', 'U') IS NULL
  CREATE TABLE USERDEVICES(...)
ELSE IF (select id from sysobjects where name='USERDEVICES' collate Chinese_Taiwan_Stroke_CS_AS) is null
BEGIN
  DROP TABLE USERDEVICES;
  CREATE TABLE USERDEVICES(...);
END;
```

若表名已存在但大小寫不符（如 `UserDevices` vs `USERDEVICES`），會先 DROP 再重建為大寫表名。這是處理早期版本表名大小寫不一致的相容邏輯。

---

## 備註

- REGID 長度 1024 字元，足以儲存 APNs Device Token（64 字元十六進位）和 FCM Registration Token（約 152 字元）。
- 此表在 EEPCore SP7 的 Web 端程式碼中無引用，操作可能由 EEP Mobile App 的 API 或獨立的推播服務處理。
- 無 infocommand 定義，無 UpdateComponent，無前端管理介面。
- 僅在 `jquery.infolight.table.system.json`（系統表列表）中出現，用於系統表清單的顯示。
