# SYS_LANGUAGE

## 用途

**系統多語系辭典表**（System Language Dictionary）。

SYS_LANGUAGE 儲存系統介面的多語系翻譯對照表，涵蓋繁中、簡中、英文、日文、韓文等語系。支援 6 種預設語系 + 2 個自訂語系。

> ⚠️ **C# 程式碼無直接引用**：後端的多語系處理由 MessageHelper 讀取 `jquery.infolight.view.json` 檔案實現，不是讀取此表。此表主要供設計端管理介面翻譯辭典，或由前端框架在特定場景下查詢。無專用 infocommand、無 Provider。

### 使用場景

| 場景 | 說明 |
|------|------|
| **設計端翻譯管理** | 管理介面翻譯辭典的 CRUD |
| **前端多語系** | 前端框架可能透過通用 DataModule 查詢此表取得翻譯 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int IDENTITY(1,1)` PK | NOT NULL | 自動遞增識別碼 |
| **IDENTIFICATION** | `nvarchar(80)` | NULL | 識別分類（如模組名稱、頁面名稱） |
| **KEYS** | `nvarchar(80)` | NULL | 翻譯鍵值（如 `btnSave`、`lblUserName`） |
| **EN** | `nvarchar(80)` | NULL | 英文翻譯 |
| **CHT** | `nvarchar(80)` | NULL | 繁體中文翻譯 |
| **CHS** | `nvarchar(80)` | NULL | 簡體中文翻譯 |
| **HK** | `nvarchar(80)` | NULL | 香港繁體翻譯 |
| **JA** | `nvarchar(80)` | NULL | 日文翻譯 |
| **KO** | `nvarchar(80)` | NULL | 韓文翻譯 |
| **LAN1** | `nvarchar(80)` | NULL | 自訂語系 1 |
| **LAN2** | `nvarchar(80)` | NULL | 自訂語系 2 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| ID | `int IDENTITY(1,1)` | `integer` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` |

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 備註

- IDENTIFICATION + KEYS 構成翻譯的邏輯鍵值，但非主鍵（只有 ID 是 PK）。
- 後端多語系主要由 `MessageHelper` 讀取 `wwwroot/json/jquery.infolight.view.json` 實現，不依賴此表。
- 僅在系統表列表 `jquery.infolight.table.system.json` 中出現。
