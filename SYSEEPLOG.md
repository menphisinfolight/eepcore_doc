# SYSEEPLOG

## 用途

**EEP 系統操作日誌表**（EEP System Operation Log）。

SYSEEPLOG 記錄系統操作日誌，包含使用者操作、執行時間、來源 IP 等資訊。由 LogHelper 寫入，支援寫入資料庫或檔案兩種模式（由設定 `logTo` 決定）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **操作稽核** | 記錄誰在何時執行了什麼操作 |
| **效能監控** | EXECUTIONTIME 記錄操作耗時（毫秒） |
| **問題排查** | 根據 CONNID / USERID / 時間範圍追蹤問題 |
| **設計端查看** | `log` infocommand 以分頁方式查詢日誌 |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPBase.Core/Utility/LogHelper.cs` | 核心寫入：先 SELECT 檢查 CONNID 是否存在，存在則 UPDATE，不存在則 INSERT |
| `EEPWebClient.Core/design/server/SystemTable.json` | `log` infocommand：`SELECT * FROM SYSEEPLOG`，keys: `CONNID`，selectPaging: true |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **CONNID** | `nvarchar(20)` PK | NOT NULL | 連線識別碼（格式：yyyyMMddHHmmssfff） |
| **LOGID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增日誌序號 |
| **LOGSTYLE** | `nvarchar(1)` | NOT NULL | 日誌風格/類型代碼（見 LogHelper.LogStyle 列舉） |
| **LOGDATETIME** | `datetime` | NOT NULL | 日誌時間 |
| **DOMAINID** | `nvarchar(30)` | NULL | 資料庫識別碼 |
| **USERID** | `varchar(20)` | NULL | 操作者帳號 |
| **LOGTYPE** | `nvarchar(1)` | NULL | 日誌類型（見 LogHelper.LogType 列舉） |
| **TITLE** | `nvarchar(64)` | NULL | 日誌標題 |
| **DESCRIPTION** | `nvarchar(max)` | NULL | 日誌詳細描述 |
| **COMPUTERIP** | `nvarchar(16)` | NULL | 客戶端 IP 位址 |
| **COMPUTERNAME** | `nvarchar(64)` | NULL | 客戶端電腦名稱 |
| **EXECUTIONTIME** | `int` | NULL | 執行耗時（毫秒） |

### LOGSTYLE 列舉（LogHelper.LogStyle）

| 值 | 說明 |
|----|------|
| 0 | System（系統） |
| 1 | Provider |
| 2 | CallMethod |
| 3 | OpenPage |
| 4 | Email |
| 5 | UserDefine |
| 6 | SSO |
| 7 | SAP |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| LOGID | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` |
| LOGDATETIME | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| DESCRIPTION | `nvarchar(max)` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |

> ⚠️ Oracle 版的主鍵只有 `LOGID`（不含 CONNID），與 SQL Server 的複合主鍵 `(CONNID, LOGID)` 不同。

---

## 主鍵

```
SQL Server / MySQL / DB2 / Informix：PRIMARY KEY (CONNID, LOGID)
Oracle：PRIMARY KEY (LOGID)
```

---

## 資料生命週期

```
寫入（logTo=db）：LogHelper
      → SELECT FROM SYSEEPLOG WHERE CONNID = @now
      → 若存在 → UPDATE（更新 DESCRIPTION、EXECUTIONTIME 等）
      → 若不存在 → INSERT（含 CONNID、LOGSTYLE、LOGDATETIME 等所有欄位）

寫入（logTo=file）：LogHelper
      → 寫入檔案而非此表
```

---

## 備註

- CONNID 以 `DateTime.Now.ToString("yyyyMMddHHmmssfff")` 產生，精確到毫秒。
- DESCRIPTION 欄位早期可能有長度限制，SQL 腳本中有 `ALTER TABLE SYSEEPLOG ALTER COLUMN DESCRIPTION nvarchar(max) NULL`。
- 當系統設定 `logTo=file` 時，日誌寫入檔案而非此表。
- `log` infocommand 啟用了 `selectPaging: true`，支援前端分頁查詢。
