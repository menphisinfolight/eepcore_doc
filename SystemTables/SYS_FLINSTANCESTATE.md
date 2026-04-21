# SYS_FLINSTANCESTATE

> 🛑 **舊版 EEP 遺留表，SP7 已完全不使用**
>
> SP7 的流程實例狀態存在 [FlowInstance](tables.html#FLOWINSTANCE) 表（CamelCase），不是這張 `SYS_FLINSTANCESTATE`。此表只剩在建表 SQL 裡殘留，C# 程式碼、前端 JS、所有 infocommand、Provider 都**沒有任何引用**（已全原始碼 grep 確認）。
>
> **請勿寫新程式讀/寫這張表**，否則會發現「啟動了流程但查不到狀態」—— 因為引擎根本沒寫進這裡。保留目的推測是早期升級相容 / 預留，實務上可以忽略。

## 用途（歷史）

**流程實例狀態表**（Flow Instance State）— 舊版 EEP 流程引擎的實例狀態儲存表。SP7 已由 `FlowInstance` 取代。

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
