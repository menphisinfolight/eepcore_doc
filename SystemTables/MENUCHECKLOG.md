# MENUCHECKLOG

## 用途

**選單簽入/簽出記錄表**（Menu Check-In/Check-Out Log）。

MENUCHECKLOG 記錄選單項目簽入簽出的歷史軌跡，包含簽出的檔案內容，用於追蹤誰在何時修改了哪個選單。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **版本控制** | 記錄選單項目的簽出/簽入歷史，支援版本回溯 |
| **變更追蹤** | 追蹤誰在何時修改了哪個選單項目 |
| **內容備份** | 儲存簽出時的檔案內容快照，可還原歷史版本 |
| **協作管理** | 防止多人同時修改同一選單項目 |
| **稽核記錄** | 提供選單變更的完整稽核軌跡 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **LOGID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增記錄識別碼 |
| **ITEMTYPE** | `nvarchar(20)` | NULL | 方案識別碼（對應 MENUITEMTYPE.ITEMTYPE） |
| **PACKAGE** | `nvarchar(50)` | NULL | 組件名稱 |
| **PACKAGEDATE** | `datetime` | NULL | 組件打包日期 |
| **FILETYPE** | `nvarchar(10)` | NULL | 檔案類型 |
| **FILENAME** | `nvarchar(60)` | NULL | 檔案名稱 |
| **FILEDATE** | `datetime` | NULL | 檔案日期 |
| **FILECONTENT** | `image` | NULL | 檔案內容（二進位） |
| **USERID** | `nvarchar(50)` | NULL | 操作者帳號（後期擴充） |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| LOGID | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` |
| FILECONTENT | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |
| PACKAGEDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |

---

## 主鍵

```
PRIMARY KEY (LOGID)
```

---

## 關聯表

### MENUTABLE（選單主表）

| 欄位 | 說明 |
|------|------|
| **CHECKOUT** | 簽出者帳號（NULL 表示未簽出） |
| **CHECKOUTDATE** | 簽出日期時間 |

### MENUITEMTYPE（方案類型表）

| 欄位 | 說明 |
|------|------|
| **ITEMTYPE** | 方案識別碼（對應 MENUCHECKLOG.ITEMTYPE） |

---

## 核心程式碼位置

| 檔案                                                      | 角色                 |
| ------------------------------------------------------- | ------------------ |
| `SystemTable.Core/UserModule.cs`                        | 選單相關資料操作           |

---

## 簽出/簽入流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   使用者    │────▶│  簽出選單   │────▶│ MENUTABLE   │
│  請求編輯   │     │             │     │ CHECKOUT=用 │
└─────────────┘     └─────────────┘     │ 戶帳號      │
                                        │ CHECKOUTDATE│
                                        │ =現在時間   │
                                        └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │MENUCHECKLOG │
                                        │ 記錄簽出    │
                                        │ FILECONTENT │
                                        │ =檔案內容   │
                                        └─────────────┘
```

### 流程說明

| 步驟 | 動作 | 說明 |
|------|------|------|
| 1 | 簽出 (Check-Out) | 使用者開始編輯選單，系統記錄於 MENUTABLE.CHECKOUT |
| 2 | 記錄備份 | 將目前檔案內容寫入 MENUCHECKLOG.FILECONTENT |
| 3 | 編輯 | 使用者編輯選單內容 |
| 4 | 簽入 (Check-In) | 使用者完成編輯，清除 MENUTABLE.CHECKOUT |

---

### SQL Server 特殊處理

SQL Server 腳本包含 USERID 欄位的後期擴充處理：

```sql
-- 若 USERID 欄位不存在則新增
IF NOT EXISTS (select * from syscolumns where id = object_id('MENUCHECKLOG') and name='USERID')
ALTER TABLE MENUCHECKLOG ADD USERID nvarchar(50) NULL
```

---

## 備註

- USERID 為後期擴充欄位，SQL Server 腳本包含相容性處理。
- FILECONTENT 使用 `image` 類型儲存簽出檔案的完整內容快照。
- 與 MENUTABLE 的 CHECKOUT/CHECKOUTDATE 欄位搭配，構成完整的版本控制體系。
- 建議定期清理過期的 MENUCHECKLOG 記錄，避免 FILECONTENT 佔用過多儲存空間。
- FILETYPE 常見值：`CS`（C# 程式碼）、`VB`（VB.NET 程式碼）、`XML`（設定檔）等。
