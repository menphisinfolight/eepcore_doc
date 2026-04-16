# SYS_FLINSTANCESTATE

## 用途

**流程實例狀態表**（Flow Instance State）。

SYS_FLINSTANCESTATE 儲存流程實例的狀態快照，包含流程實例 ID、二進位序列化的狀態內容、狀態碼和資訊。

> ⚠️ **舊版遺留，程式碼無引用**：此表在 SQL 建表腳本中有定義，但 EEPCore SP7 的 C# 程式碼和前端 JavaScript 均無引用。無對應的 infocommand、無 Provider。新版流程引擎使用 FLOWINSTANCE 表儲存流程實例狀態，此表為舊版流程引擎的殘留結構。

### 與 FLOWINSTANCE 的關係

| 項目 | SYS_FLINSTANCESTATE（舊） | FLOWINSTANCE（現行） |
|------|--------------------------|---------------------|
| **狀態儲存** | STATE (image) | State (image) |
| **主鍵** | FLINSTANCEID | InstanceID |
| **額外欄位** | STATUS (int), INFO | Datetime, Parameter, ProjectID |
| **程式碼引用** | 無 | FlowInstanceHelper 完整 CRUD |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FLINSTANCEID** | `nvarchar(50)` PK | NOT NULL | 流程實例識別碼 |
| **STATE** | `image` | NOT NULL | 流程狀態內容（二進位序列化） |
| **STATUS** | `int` | NULL | 狀態碼 |
| **INFO** | `nvarchar(200)` | NULL | 狀態資訊 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 |
|------|-----------|--------|-------|-----|
| STATE | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |

---

## 主鍵

```
PRIMARY KEY (FLINSTANCEID)
```

---

## 備註

- 與 SYS_FLDEFINITION（舊版流程定義表）同屬舊版流程引擎的系統表，兩者在 EEPCore SP7 中均無程式碼引用。
- 新版流程引擎的實例狀態存放在 FLOWINSTANCE.State 欄位，以 BinaryFormatter 序列化整個 Instance 物件。
- 僅在系統表列表 `jquery.infolight.table.system.json` 中出現。
