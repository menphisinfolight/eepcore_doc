# SYSSQLLOG

## 用途

**SQL 執行日誌表**（SQL Execution Log）。

SYSSQLLOG 記錄系統執行的 SQL 語句日誌，用於 SQL 稽核、效能分析、問題排查。記錄誰執行了什麼 SQL、何時執行、開發者身份等。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **SQL 效能分析** | 分析慢查詢、優化 SQL 執行效率 |
| **問題排查** | 追蹤錯誤發生時執行的 SQL 語句 |
| **稽核追蹤** | 記錄誰在何時執行了什麼操作 |
| **開發除錯** | 開發階段檢視實際執行的 SQL |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **LOGSTYLE** | `nvarchar(1)` | NULL | 日誌風格/類型代碼 |
| **LOGDATETIME** | `datetime` | NULL | 執行時間 |
| **USERID** | `varchar(20)` | NULL | 執行者帳號 |
| **DEVELOPERID** | `varchar(20)` | NULL | 開發者帳號（設計模式下操作者） |
| **DESCRIPTION** | `text` | NULL | SQL 描述 |
| **SQLSENTENCE** | `nvarchar(max)` | NULL | 完整 SQL 語句 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| LOGDATETIME | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| DESCRIPTION | `text` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |
| SQLSENTENCE | `nvarchar(max)` | `clob` | `text` | `CLOB (100M)` / `LVARCHAR(9800)` |

---

## 日誌記錄機制

### 檔案模式（預設）

根據 `LogHelper.SqlLog()` 實作：

```csharp
public void SqlLog(JObject value)
{
    var config = GetConfig();
    if (ClientInfo != null && config["logSqlTo"]?.ToString() == "file" && !ClientInfo.Dev)
    {
        var date = DateTime.Now;
        value["user"] = ClientInfo.User;
        LogFile(date.ToString("yyyyMMdd") + ".sqllog", JsonConvert.SerializeObject(value) + ",");
    }
}
```

**記錄位置**：`design/log/YYYYMMDD.sqllog`

**記錄內容**（JSON 格式）：

| 欄位 | 說明 |
|------|------|
| **database** | 資料庫名稱 |
| **sql_str** | SQL 語句（空白正規化） |
| **dt1** | 開始時間（1970/1/1 起的 ticks/10000） |
| **dt2** | 結束時間（1970/1/1 起的 ticks/10000） |
| **err** | 錯誤訊息（若有） |
| **count** | 影響筆數 |
| **user** | 執行者帳號 |

### 觸發時機

根據 `DatabaseHelper.cs`：

```csharp
// 每次執行 SQL 後記錄
new LogHelper(ClientInfo).SqlLog(value);
LogHelper.DebuggerLog($"sqllog@{Name}", value);
```

**觸發點**：
- `ExecuteNonQuery()` - 執行 INSERT/UPDATE/DELETE
- `ExecuteScalar()` - 執行查詢

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPBase.Core/Utility/LogHelper.cs` | SQL 日誌寫入與讀取 |
| `EEPBase.Core/Utility/DatabaseHelper.cs` | 執行 SQL 時觸發記錄 |
| `EEPGlobal.Core/Provider/LogProvider.cs` | 提供 SQL 日誌查詢 API |

---

## 日誌查詢與匯出

### 查詢日誌

```csharp
// LogProvider.cs
public object GetSqlLogs(EEPBase.Core.ClientInfo clientInfo)
{
    var logs = LogHelper.GetDebuggerLogs($"sqllog@{clientInfo.Database}");
    return JsonConvert.SerializeObject(logs);
}
```

### 匯出 Excel

```csharp
// LogProvider.cs
public object ExportSqlLogs(QueryOptions options, ...)
{
    var columns = new JArray(){
        new JObject{ { "field", "dt1" }, { "title", "時間" } },
        new JObject{ { "field", "database" }, { "title", "資料庫" } },
        new JObject{ { "field", "user" }, { "title", "使用者" } },
        new JObject{ { "field", "sql_str" }, { "title", "SQL" } },
        new JObject{ { "field", "dt2" }, { "title", "耗時(ms)" } },
        new JObject{ { "field", "count" }, { "title", "筆數" } }
    };
    return new FileProvider(context).ExportToExcel(datas, columns, "");
}
```

---

## 備註

- 此表無明確主鍵定義，純日誌用途。
- SQLSENTENCE 使用 `nvarchar(max)` 儲存完整 SQL，不受長度限制。
- DEVELOPERID 與 USERID 的區別：USERID 是實際登入帳號，DEVELOPERID 是設計模式下的開發者帳號。
- 預設使用檔案模式記錄（`design/log/*.sqllog`），非資料庫寫入。
- 開發者模式（`ClientInfo.Dev = true`）不會記錄 SQL 日誌。
- 可透過設定 `logSqlTo` 切換記錄方式。
