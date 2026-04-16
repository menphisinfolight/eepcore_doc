# SYSERRLOG

## 用途

**系統錯誤日誌表**（System Error Log）。

SYSERRLOG 記錄系統發生的錯誤，包含錯誤訊息、堆疊追蹤、畫面截圖、處理狀態等。支援錯誤追蹤生命週期：從發現到處理完成。

> ⚠️ **C# 程式碼無直接引用**：此表在 SQL 建表腳本中有完整定義，但 EEPCore SP7 的 C# 程式碼中未找到 INSERT 或操作此表的邏輯。推測由系統底層（如全域例外處理中間件）或管理介面透過 DataModule 通用框架操作。

### 預期使用場景

| 場景 | 說明 |
|------|------|
| **錯誤記錄** | 系統發生例外時自動寫入（ERRMESSAGE + ERRSTACK + ERRDATE） |
| **錯誤排查** | 開發者查看 ERRSTACK（堆疊追蹤）和 ERRSCREEN（畫面截圖）排查問題 |
| **錯誤追蹤** | 設定 OWNER（處理者）、PROCESSDATE（處理日期）、PRODESCRIP（處理說明） |
| **狀態管理** | STATUS 欄位追蹤處理進度 |

### 錯誤處理流程

```
發生錯誤 → 寫入 SYSERRLOG（STATUS=待處理）
         → 開發者查看 ERRSTACK / ERRSCREEN 排查
         → 設定 OWNER / PROCESSDATE / PRODESCRIP
         → 更新 STATUS 為已處理
```

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ERRID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增錯誤識別碼 |
| **USERID** | `varchar(20)` | NULL | 發生錯誤時的使用者帳號 |
| **MODULENAME** | `nvarchar(30)` | NULL | 發生錯誤的模組名稱 |
| **ERRMESSAGE** | `nvarchar(255)` | NULL | 錯誤訊息 |
| **ERRSTACK** | `text` | NULL | 錯誤堆疊追蹤（完整 call stack） |
| **ERRDESCRIP** | `nvarchar(255)` | NULL | 錯誤描述 |
| **ERRDATE** | `datetime` | NULL | 錯誤發生時間 |
| **ERRSCREEN** | `image` | NULL | 錯誤畫面截圖（二進位） |
| **OWNER** | `nvarchar(20)` | NULL | 處理者 USERID |
| **PROCESSDATE** | `datetime` | NULL | 處理日期 |
| **PRODESCRIP** | `nvarchar(255)` | NULL | 處理說明 |
| **STATUS** | `nvarchar(2)` | NULL | 處理狀態代碼 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| ERRID | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` |
| ERRSTACK | `text` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |
| ERRSCREEN | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |
| ERRDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| PROCESSDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |

---

## 主鍵

```
PRIMARY KEY (ERRID)
```

---

## 備註

- ERRSCREEN 使用 `image`/`BLOB` 類型儲存畫面截圖，可用於重現錯誤場景。
- ERRSTACK 使用 `text`/`clob` 儲存完整堆疊追蹤，長度不受限。
- STATUS 欄位長度 `nvarchar(2)`，使用代碼表示不同狀態（具體值未在程式碼中定義）。
- 僅在系統表列表 `jquery.infolight.table.system.json` 中出現，無專用的 infocommand 或 Provider。
