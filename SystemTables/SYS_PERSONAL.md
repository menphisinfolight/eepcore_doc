# SYS_PERSONAL

## 用途

**使用者個人化設定表**（User Personalization Settings）。

SYS_PERSONAL 儲存使用者對表單元件的個人化偏好設定，如欄位順序、欄位寬度、排序條件、篩選狀態等。每個使用者對同一表單的同一元件可以有獨立的 UI 偏好。

> ⚠️ **C# 程式碼無直接引用**：此表在 SQL 建表腳本中有完整定義，但 EEPCore SP7 的 C# 程式碼中未找到直接引用 `SYS_PERSONAL` 的操作。推測由前端框架透過 DataModule 通用機制存取。

### ���期使用場景

| 場景 | 說明 |
|------|------|
| **DataGrid 欄位配置** | 使用者調整欄位順序、寬度、顯示/隱藏後自動儲存 |
| **查詢條件記憶** | ��存使用者常用的查詢條���組合 |
| **排序偏好** | 記錄使用者偏好的資料排序方式 |
| **跨裝置同步** | 個人化設定跟隨使用者帳號，在不同裝置登入時保持一致 |

### 關聯表

```
SYS_PERSONAL ──(USERID)──> USERS      （使用者帳號）
             ──(FORMNAME)─> MENUTABLE  （表單/選單識別）
```

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FORMNAME** | `nvarchar(60)` PK | NOT NULL | 表單名稱（對應 MENUTABLE.MENUID 或頁面識別碼） |
| **COMPNAME** | `nvarchar(30)` PK | NOT NULL | 元件名��（如 DataGrid、Form 等前端元件 ID） |
| **USERID** | `nvarchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **REMARK** | `nvarchar(30)` | NULL | 備註 |
| **PROPCONTENT** | `ntext` | NULL | 個人化設定內容（序列化的 UI 偏好） |
| **CREATEDATE** | `datetime` | NULL | 建立日期 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| PROPCONTENT | `ntext` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |
| CREATEDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |

---

## 主鍵

```
PRIMARY KEY (FORMNAME, COMPNAME, USERID)
```

同一使用者在同一表單的同一元件上只有一筆個人化設定。

---

## 備註

- PROPCONTENT 使用 `ntext`/`clob` 儲存序列化設定，內容可能為 JSON 格式（欄位順序、寬度、顯示狀態等）。
- 此表實現了「同一系統，不同使用者看到不同的 UI 排列」的個人化功���。
- 清除個人化設定可讓使用者恢復系統預設的 UI 配置。
- 僅在系統表列表 `jquery.infolight.table.system.json` 中出現，無專用 infocommand。
