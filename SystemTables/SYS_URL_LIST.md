# SYS_URL_LIST

## 用途

**短網址對應表（設計師方案版）**（Short URL Mapping by Designer/Solution/DBAlias）。

SYS_URL_LIST 以設計師帳號 + 方案 + 資料庫別名為三維鍵值，儲存對應的短網址。這是較早期的短網址映射方式，在 SQL Server / Oracle / MySQL 已被 SYS_URL_LIST2（以完整 URL 為鍵）取代，但 **DB2 和 Informix 仍在使用此表**。

### 使用場景

| 場景 | 說明 |
|------|------|
| **短網址查詢（DB2/Informix）** | `urlList` infocommand 在 DB2/Informix 版指向 `SELECT * FROM SYS_URL_LIST`，C# 程式碼透過 DataModule 查詢 |
| **短網址產生（DB2/Informix）** | AccountProvider / MenuProvider 查詢短網址時，若找不到則透過 pics.ee API 產生並寫入 |

### 與 SYS_URL_LIST2 的關係

| 項目 | SYS_URL_LIST | SYS_URL_LIST2 |
|------|-------------|---------------|
| **主鍵** | Designer + Solution + DBAlias | URL（完整網址） |
| **欄位數** | 4 | 2 |
| **使用資料庫** | DB2、Informix | SQL Server、Oracle、MySQL |
| **infocommand** | `urlList`（DB2/Informix 版 SystemTable JSON） | `urlList`（SQL Server/Oracle/MySQL 版 SystemTable JSON） |

> ⚠️ `urlList` 這個 infocommand ID 在不同資料庫的 SystemTable JSON 中指向不同的表：
> - `SystemTable.json`（SQL Server）→ `SYS_URL_LIST2`
> - `SystemTable.oracle.json` → `SYS_URL_LIST2`
> - `SystemTable.mysql.json` → `SYS_URL_LIST2`
> - `SystemTable.db2.json` → `SYS_URL_LIST`
> - `SystemTable.informix.json` → `SYS_URL_LIST`

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | 短網址查詢/產生：`GetShortUrl()` 透過 `DataModule.GetDataset("urlList")` 查詢，若無結果則呼叫 pics.ee API 產生並 INSERT |
| `EEPGlobal.Core/Provider/MenuProvider.cs` | 同上，邏輯與 AccountProvider 幾乎相同 |
| `EEPWebClient.Core/design/server/SystemTable.db2.json` | DB2 的 `urlList` infocommand：`SELECT * FROM SYS_URL_LIST` |
| `EEPWebClient.Core/design/server/SystemTable.informix.json` | Informix 的 `urlList` infocommand：`SELECT * FROM "SYS_URL_LIST"` |
| `SystemTable.Core/UserModule.cs` | `ucUrlList` UpdateComponent（通用，無特殊事件處理） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **Designer** | `varchar(20)` PK | NOT NULL | 設計師帳號 |
| **Solution** | `nvarchar(20)` PK | NOT NULL | 方案識別碼（對應 MENUITEMTYPE.ITEMTYPE） |
| **DBAlias** | `nvarchar(50)` PK | NOT NULL | 資料庫別名 |
| **ShortURL** | `nvarchar(50)` | NOT NULL | 對應的短網址 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar/nvarchar`。

---

## 主鍵

```
PRIMARY KEY (Designer, Solution, DBAlias)
```

---

## 資料生命週期（DB2/Informix 環境）

```
查詢：AccountProvider.GetShortUrl(url) / MenuProvider.GetShortUrl(url)
      → DataModule.GetDataset("urlList", WHERE URL = @url)
      → 若有結果 → 回傳 ShortURL

產生：查無結果時
      → 呼叫 pics.ee API 產生短網址
      → DataModule.UpdateDataset("urlList", INSERT {ShortURL: 短網址})
      → 回傳短網址
```

---

## 備註

- C# 程式碼（AccountProvider / MenuProvider）以 `"urlList"` 為 infocommand ID 操作，實際指向哪張表由各資料庫版本的 SystemTable JSON 決定。
- DB2/Informix 環境下，C# 程式碼查詢時使用 `URL` 欄位作為條件，但 SYS_URL_LIST 實際上沒有 `URL` 欄位（只有 Designer/Solution/DBAlias/ShortURL），這可能是一個相容性問題或由 DataModule 層做了轉換。
- 短網址產生使用外部服務 pics.ee API。
- SQL Server / Oracle / MySQL 環境下此表仍會被建立（SQL 腳本中有 CREATE TABLE），但不會被程式碼存取。
