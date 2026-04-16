# SYS_REPORT

## 用途

**報表定義表**（Report Definition）。

SYS_REPORT 儲存 EEP Core 報表設計器的報表定義，包含版面配置、字型設定、資料來源、參數、郵件設定等。報表的所有設定以二進位（image / BLOB）格式儲存。

> ⚠️ **C# 程式碼無直接引用表名**：此表在 SQL 建表腳本中有完整定義，但 C# 和 JS 程式碼中未找到直接引用 `SYS_REPORT` 的操作。推測由 DataModule 通用框架或報表引擎（EEPReport.Core）透過動態方式存取。

### 使用場景

| 場景 | 說明 |
|------|------|
| **報表設計** | 報表設計器讀取/寫入報表定義（版面、字型、欄位佈局等） |
| **報表產生** | 執行報表時讀取設定，渲染 PDF / Excel / HTML 輸出 |
| **郵件排程** | MAILSETTING 定義報表排程發送的郵件設定 |
| **多資料來源** | DATASOURCES / DATASOURCE_PROVIDER 支援跨表/跨資料庫查詢 |

### 與其他報表表的區分

| 資料表 | 報表類型 | 設計工具 |
|--------|----------|----------|
| **SYS_REPORT** | 原生格線報表 | EEP 報表設計器 |
| **SYS_DOCFILES** | Word 報表 | Word 範本 |
| **SYS_XLSFILES** | Excel 報表 | Excel 範本 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **REPORTID** | `nvarchar(50)` PK | NOT NULL | 報表唯一識別碼 |
| **FILENAME** | `nvarchar(50)` PK | NOT NULL | 報表檔案名稱 |
| **REPORTNAME** | `nvarchar(50)` | NULL | 報表顯示名稱 |
| **DESCRIPTION** | `nvarchar(50)` | NULL | 報表描述 |
| **FILEPATH** | `nvarchar(50)` | NULL | 報���實體檔案存放路徑 |
| **OUTPUTMODE** | `nvarchar(20)` | NULL | 輸出模式（PDF / Excel / HTML / Printer） |
| **HEADERREPEAT** | `nvarchar(5)` | NULL | 表頭是否每頁重複（Y/N） |
| **HEADERFONT** | `image` | NULL | 表頭字型設定（二進位序列化） |
| **HEADERITEMS** | `image` | NULL | 表頭項目佈局 |
| **FOOTERFONT** | `image` | NULL | 表尾字型設定 |
| **FOOTERITEMS** | `image` | NULL | 表尾項目佈局 |
| **FIELDFONT** | `image` | NULL | 資料區字型設定 |
| **FIELDITEMS** | `image` | NULL | 資料區欄位佈局 |
| **SETTING** | `image` | NULL | 報表整體設定（紙張、邊界、群組條件等） |
| **FORMAT** | `image` | NULL | 格式化規則（數字、日期、貨幣格式） |
| **PARAMETERS** | `image` | NULL | 報表執行參數 |
| **IMAGES** | `image` | NULL | 報表內嵌圖片（Logo、浮水印等） |
| **MAILSETTING** | `image` | NULL | 郵件發送設定 |
| **DATASOURCE_PROVIDER** | `nvarchar(50)` | NULL | 資料來源提供者名稱 |
| **DATASOURCES** | `image` | NULL | 多資料來源定義（跨庫連線與 SQL） |
| **CLIENT_QUERY** | `image` | NULL | 前端額外查詢條件 |
| **REPORT_TYPE** | `nvarchar(1)` | NULL | 報表類型代碼 |
| **TEMPLATE_DESC** | `nvarchar(50)` | NULL | 範本說明或版本標記 |

### 跨資料庫差異

| 欄位類型 | SQL Server | Oracle | MySQL | DB2 / Informix |
|----------|-----------|--------|-------|----------------|
| image 系列 | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |

---

## 主鍵

```
PRIMARY KEY (REPORTID, FILENAME)
```

複合主鍵允許同一 REPORTID 下存在多個 FILENAME（如不同語系/版本/輸出格式的報表）。

---

## 備註

- ��表大量使用二進位欄位（image/BLOB），儲存序列化的報表設定物件。
- DATASOURCES 支援多資料庫連線，可在同一張報表裡 JOIN 不同系統的資料表。
- CLIENT_QUERY 供前端覆蓋或補充後端 DATASOURCES 的查詢條件。
- 僅在系統表列表中出現，推測由 EEPReport.Core 模組透過動態方式��取。
