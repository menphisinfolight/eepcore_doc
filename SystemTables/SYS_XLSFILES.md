# SYS_XLSFILES

## 用途

**Excel 報表定義表**（Excel Report Definition）。

SYS_XLSFILES 儲存 Excel 報表的定義，包含查詢格式、報表格式、表頭格式等。與 SYS_DOCFILES（Word 報表）為不同的報表體系。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **Excel 報表設計** | 定義 Excel 格式的報表範本 |
| **資料匯出** | 將查詢結果匯出為 Excel 檔案 |
| **套表列印** | 套印預設格式的 Excel 報表 |
| **動態報表產生** | 根據定義動態產生 Excel 內容 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FILENAME** | `nvarchar(80)` PK | NOT NULL | 報表定義檔案名稱 |
| **NAME** | `nvarchar(50)` | NOT NULL | 報表顯示名稱 |
| **MASTERTABLE** | `nvarchar(50)` | NOT NULL | 主檔資料表名稱 |
| **MASTERCOMMAND** | `nvarchar(50)` | NULL | 主檔命令名稱 |
| **KEYFIELDS** | `nvarchar(100)` | NULL | 關聯鍵值欄位 |
| **NAMEFIELDS** | `nvarchar(100)` | NULL | 名稱欄位 |
| **FIELDMAX** | `int` | NULL | 欄位數上限 |
| **QUERYFORMAT** | `nvarchar(max)` | NULL | 查詢條件格式定義（JSON） |
| **REPORTFORMAT** | `nvarchar(max)` | NULL | 報表格式定義（JSON） |
| **HEADFORMAT** | `nvarchar(max)` | NULL | 表頭格式定義（JSON） |
| **XLSDATE** | `datetime` | NULL | 報表定義日期 |
| **OWNER** | `nvarchar(50)` | NULL | 擁有者 |
| **TYPE** | `nvarchar(50)` | NULL | 報表類型（如 `excel_edit`） |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL |
|------|-----------|--------|-------|
| QUERYFORMAT | `nvarchar(max)` | `clob` | `text` |
| REPORTFORMAT | `nvarchar(max)` | `clob` | `text` |
| HEADFORMAT | `nvarchar(max)` | `clob` | `text` |
| XLSDATE | `datetime` | `date` | `datetime` |

---

## 主鍵

```
PRIMARY KEY (FILENAME)
```

---

## 報表類型（TYPE）

| 類型 | 說明 |
|------|------|
| **excel** | 一般 Excel 報表 |
| **excel_edit** | 可編輯的 Excel 報表 |

---

## 格式欄位 JSON 結構

### QUERYFORMAT（查詢條件格式）

定義報表的查詢條件介面：

```json
[
  {
    "FIELD_NAME": "欄位名稱",
    "CAPTION": "顯示標題",
    "CONDITION": "=",
    "ANDOR": "AND"
  }
]
```

### REPORTFORMAT（報表格式定義）

定義報表的欄位配置：

```json
[
  {
    "FIELD_NAME": "欄位名稱",
    "CAPTION": "顯示標題",
    "WIDTH": 100,
    "ALIGNMENT": "left",
    "FORMAT": "",
    "TOTAL": "none"
  }
]
```

### HEADFORMAT（表頭格式定義）

定義報表表頭的額外資訊：

```json
[
  {
    "FIELD_NAME": "欄位名稱",
    "CAPTION": "顯示標題",
    "WIDTH": 100
  }
]
```

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | Excel 報表資料操作（ucSYS_XLSFILES_onBeforeInsert） |
| `EEPGlobal.Core/Provider/ExcelProvider.cs` | Excel 報表產生與處理 |
| `EEPGlobal.Core/Provider/UserTableProvider.cs` | Excel 報表資料讀取 |
| `EEPGlobal.Core/Provider/FileListProvider.cs` | 檔案清單管理 |
| `EEPServerTools.Core/Utility/ParserHelper.cs` | Excel 報表解析 |

---

## Excel 報表產生流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ SYS_XLSFILES│────▶│ 讀取報表    │────▶│ 解析格式    │
│ 報表定義    │     │ 定義        │     │ JSON        │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌─────────────┐           ▼
                    │  產生 Excel │◀────┌─────────────┐
                    │  檔案       │     │ 套用資料    │
                    └─────────────┘     │ 與格式      │
                                        └─────────────┘
```

---

## 與 SYS_DOCFILES 的關係

| 比較項目 | SYS_XLSFILES | SYS_DOCFILES |
|----------|--------------|--------------|
| **用途** | Excel 報表 | Word 報表 |
| **檔案格式** | .xlsx | .docx |
| **資料結構** | 表格導向 | 文件導向 |
| **主要欄位** | QUERYFORMAT/REPORTFORMAT | TEMPLATECONTENT |

---

## 新增報表處理

根據 `UserModule.cs`：

```csharp
public bool ucSYS_XLSFILES_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    // 刪除同名舊報表
    sqls.Add($"DELETE FROM SYS_XLSFILES WHERE FILENAME = {value}");
}
```

**說明**：新增報表時會先刪除同名舊報表，確保 FILENAME 唯一性。

---

## 備註

- QUERYFORMAT、REPORTFORMAT、HEADFORMAT 使用 `nvarchar(max)` 儲存 JSON 格式的設定。
- MASTERCOMMAND、TYPE、HEADFORMAT 為後期擴充欄位。
- Excel 報表檔案實體儲存於 `design/files/{FILENAME}.xlsx`。
- 與 Word 報表（SYS_DOCFILES）為獨立的報表體系。
