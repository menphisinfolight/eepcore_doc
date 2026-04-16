# LogProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/LogProvider.cs` |
| 行數 | 405 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `BaseProvider` |

## 用途

系統日誌管理 Provider，支援載入/儲存日誌設定、查詢與匯出系統日誌、SQL 日誌、Server 日誌及 Chat 日誌。日誌儲存方式可切換為 database 或 file。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 取得日誌設定（`log.cfg`） |
| `save` | 儲存日誌設定 |
| `view` | 查詢日誌（支援分頁與條件篩選） |
| `viewSql` | 查詢 SQL 日誌 |
| `getSqlLogs` | 取得 debugger SQL 日誌 |
| `getServerLogs` | 取得 Server 日誌 |
| `getChatLogs` | 取得 Chat 日誌 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetLogs(options, clientInfo)` | 依 `logTo` 設定從 database 或 file 讀取日誌，支援分頁 |
| `ExportLogs(...)` | 匯出日誌為 Excel |
| `ExportSqlLogs(...)` | 匯出 SQL 日誌為 Excel |
| `GetLogFiles(dir, ext, whereItems)` | 依日期範圍篩選 `.log` 檔案 |
| `GetLogLines(data)` | 解析 log 檔內容，以分號分隔欄位，去重複 |

## 日誌型別 (LOGTYPE)

| 值 | 說明 |
|----|------|
| 0 | Normal |
| 2 | Unknown |
| 3 | Error |

## 日誌樣式 (LOGSTYLE)

| 值 | 說明 |
|----|------|
| 0 | System |
| 1 | Provider |
| 2 | CallMethod |
| 3 | OpenMenu |
| 4 | Email |
| 5 | UserDefine |
| 6 | SSO |
| 7 | CallProc |

## 備註

- 檔案日誌格式：每行以分號分隔 10 個欄位（CONNID;LOGSTYLE;LOGDATETIME;DOMAINID;USERID;LOGTYPE;TITLE;DESCRIPTION;COMPUTERIP;EXECUTIONTIME）
- debugger 日誌鍵名格式：`sqllog@{database}`、`serverlog`、`chatlog`
