# SYS_MESSENGER

## 用途

**系統訊息傳遞表**（System Messenger / Internal Messaging）。

SYS_MESSENGER 儲存系統內的訊息通知，支援使用者之間或系統對使用者的訊息推播。包含訊息內容、發送者、接收者、發送時間、已讀狀態等。

### 使用場景

| 場景 | 說明 |
|------|------|
| **未讀訊息數** | `myUnreadMsgCount` infocommand：`SELECT COUNT(*) FROM SYS_MESSENGER WHERE STATUS != 'Y'` |
| **訊息清單** | `SYS_MESSENGER` infocommand：LEFT JOIN USERS 取得發送者和接收者姓名，以 secField=USERID 過濾 |
| **標記已讀** | `UserModule.msgConfirm(ids)`：`UPDATE SYS_MESSENGER SET STATUS='Y', RECTIME=@now WHERE id IN (@ids)` |
| **全部已讀** | `UserModule.msgRead(userID)`：`UPDATE SYS_MESSENGER SET STATUS='Y' WHERE userid=@userID` |
| **刪除訊息** | `UserModule.msgDel(ids)`：`DELETE FROM SYS_MESSENGER WHERE id IN (@ids)` |

### 關聯表

```
SYS_MESSENGER ──(USERID)───> USERS  （接收者，LEFT JOIN 取 USERNAME）
              ──(SENDERID)─> USERS  （發送者，LEFT JOIN 取 SENDER_NAME）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `msgConfirm()`：標記已讀（UPDATE STATUS='Y' + RECTIME）；`msgDel()`：刪除；`msgRead()`：全部已讀 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `myUnreadMsgCount`：未讀數；`SYS_MESSENGER`：訊息清單（含 JOIN USERS） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增訊息識別碼（後期新增） |
| **USERID** | `varchar(20)` | NOT NULL | 接收者帳號 |
| **MESSAGE** | `nvarchar(1024)` | NULL | 訊息內容 |
| **PARAS** | `nvarchar(max)` | NULL | 訊息參數（JSON，用於動態訊息模板） |
| **SENDTIME** | `nvarchar(14)` | NULL | 發送時間（格式：yyyyMMddHHmmss） |
| **SENDERID** | `nvarchar(20)` | NULL | 發送者帳號 |
| **RECTIME** | `nvarchar(14)` | NULL | 已讀時間（格式：yyyyMMddHHmmss） |
| **STATUS** | `char(1)` | NULL | 訊息狀態（`Y`=已讀，空/非Y=未讀） |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| ID | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` |
| PARAS | `nvarchar(max)` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 備註

- ID 為後期新增的主鍵，SQL 腳本包含 `ALTER TABLE SYS_MESSENGER ADD ID int IDENTITY(1,1) NOT NULL CONSTRAINT PK_SYS_MESSENGER PRIMARY KEY CLUSTERED` 的相容處理。
- MESSAGE 長度從舊版擴展到 `nvarchar(1024)`（SQL 腳本有 ALTER TABLE）。
- SENDTIME / RECTIME 使用字串格式 `nvarchar(14)`（yyyyMMddHHmmss），而非 datetime。
- `msgConfirm` 的 SQL 使用字串拼接（`where id in (` + ids + `)`），有潛在 SQL Injection 風險。
- `myUnreadMsgCount` 查詢條件為 `STATUS != 'Y' AND STATUS != 'y'`，同時處理大小寫。
