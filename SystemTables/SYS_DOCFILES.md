# SYS_DOCFILES

## 用途

**Word 報表定義表**（Word Document Report Definition）。

SYS_DOCFILES 儲存 Word 報表的定義，包含主檔/明細表的對應關係、欄位對應、報表格式設定等。是 EEP Word 報表系統的核心配置表。

### 使用場景

| 場景 | 說明 |
|------|------|
| **Word 報表產生** | WordProvider / ParserHelper 讀取 SYS_DOCFILES 取得主明細表對應和格式設定 |
| **報表設計** | 設計端透過 `SYS_DOCFILES` infocommand 管理報表定義（nonlogon: true，無需登入） |
| **INSERT 先刪後寫** | `ucSYS_DOCFILES_onBeforeInsert()`：INSERT 前先 DELETE 同 FILENAME 記錄 |
| **DELETE 連動 COLDEF** | `ucSYS_DOCFILES_onBeforeDelete()`：刪除報表時一併刪除 COLDEF 中對應的欄位定義 |
| **報表檔案清單** | `SYS_FILES` infocommand：UNION SYS_DOCFILES + SYS_XLSFILES 合併顯示所有報表 |

### 關聯表

```
SYS_DOCFILES ──(MASTERTABLE/DETAILTABLE)──> COLDEF（欄位定義）
             ──(FILENAME)────────────────> SYS_FILES（UNION SYS_XLSFILES 合併清單）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/WordProvider.cs` | Word 報表產生：讀取 SYS_DOCFILES + COLDEF，產生 Word 文件 |
| `EEPGlobal.Core/Provider/ExcelProvider.cs` | Excel 報表產生時也可能引用 |
| `EEPServerTools.Core/Utility/ParserHelper.cs` | 報表解析：讀取欄位對應 |
| `SystemTable.Core/UserModule.cs` | `ucSYS_DOCFILES_onBeforeInsert()`：先刪後寫；`ucSYS_DOCFILES_onBeforeDelete()`：連動刪除 COLDEF |
| `EEPGlobal.Core/Provider/MenuProvider.cs` | 選單相關報表操作 |
| `EEPGlobal.Core/Provider/FileListProvider.cs` | 報表檔案清單 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `SYS_DOCFILES`（nonlogon: true）；`SYS_FILES`（UNION 合併清單） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FILENAME** | `nvarchar(80)` PK | NOT NULL | 報表定義檔案名稱 |
| **NAME** | `nvarchar(50)` | NOT NULL | 報表顯示名稱 |
| **MASTERTABLE** | `nvarchar(50)` | NOT NULL | 主檔資料表名稱 |
| **DETAILTABLE** | `nvarchar(50)` | NULL | 明細表 1 名稱 |
| **DETAIL2TABLE** | `nvarchar(50)` | NULL | 明細表 2 名稱（後期擴充） |
| **DETAIL3TABLE** | `nvarchar(50)` | NULL | 明細表 3 名稱（後期擴充） |
| **SUBDETAILTABLE** | `nvarchar(50)` | NULL | 子明細表名稱（後期擴充） |
| **KEYFIELDS** | `nvarchar(100)` | NULL | 關聯鍵值欄位 |
| **NAMEFIELDS** | `nvarchar(100)` | NULL | 名稱欄位 |
| **FIELDMAX** | `int` | NULL | 主檔欄位數上限 |
| **DETAILFIELDMAX** | `int` | NULL | 明細欄位數上限 |
| **REPORTFORMAT** | `nvarchar(max)` | NULL | 報表格式定義（JSON 序列化的欄位對應） |
| **DOCDATE** | `datetime` | NULL | 報表定義日期 |
| **OWNER** | `nvarchar(50)` | NULL | 擁有者 |

### 跨資料庫差異

⚠️ **DB2 新裝缺 `SUBDETAILTABLE`**（第三層明細失效）。
⚠️ DB2 / Informix `REPORTFORMAT` 上限 8000 字元，大型報表定義可能超出。

👉 升級 ALTER 支援矩陣、手動補欄位 SQL：**問題_SP7_跨資料庫欄位差異盤點**

---

## 主鍵

```
PRIMARY KEY (FILENAME)
```

---

## 資料生命週期

```
新增：設計端 → ucSYS_DOCFILES_onBeforeInsert()
      → DELETE FROM SYS_DOCFILES WHERE FILENAME = @filename（先刪後寫）
      → INSERT INTO SYS_DOCFILES

刪除：設計端 → ucSYS_DOCFILES_onBeforeDelete()
      → DELETE FROM COLDEF WHERE TABLE_NAME IN (@tableNames)（連動刪除欄位定義）
      → DELETE FROM SYS_DOCFILES WHERE FILENAME = @filename
```

---

## 備註

- 支援最多 3 個明細表 + 1 個子明細表，實現主明細多層結構。
- DETAIL2TABLE、DETAIL3TABLE、SUBDETAILTABLE 為後期擴充欄位（SQL 腳本中有 ALTER TABLE）。
- 刪除 SYS_DOCFILES 時會一併刪除 COLDEF 中對應的記錄，確保報表與欄位定義一致。
- Oracle 版本的 `ucSYS_DOCFILES_onBeforeInsert_Oracle` 有特殊處理（手動組 INSERT SQL，return false）。
- `SYS_DOCFILES` infocommand 設定 `nonlogon: true`，允許未登入狀態下存取。
