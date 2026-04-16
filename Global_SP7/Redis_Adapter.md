# Redis (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/Redis.cs` |
| 行數 | 123 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

Redis 日誌與偵錯緩衝 Provider，實作 `EEP.Flow.ILogInfoProvider` / `IDebugProvider` 及 `EEP.Processor` 對應介面。**目前 Redis 連線已全部註解，改用記憶體字典（`Buffers`）替代。**

## 主要方法

| 方法 | 說明 |
|------|------|
| `LoadBuffer(key)` | 從記憶體字典載入 Base64 緩衝資料 |
| `SaveBuffer(key, buffer)` | 儲存 Base64 緩衝資料至記憶體字典 |
| `RemoveBuffer(key)` | 移除緩衝資料 |
| `LogSql(commandText, dt, e)` | SQL 日誌記錄（目前為空實作） |
| `Enable` | 是否啟用日誌（目前固定回傳 `false`） |

## 備註

- Redis 相關程式碼全部以註解保留，包含 `ConnectionMultiplexer.Connect` / `StringGet` / `ListRightPush` 等
- 記憶體字典操作使用 `lock(typeof(RedisLogProvider))` 確保執行緒安全
- 此類別同時服務 Flow 和 Processor 兩套引擎的日誌與偵錯需求
