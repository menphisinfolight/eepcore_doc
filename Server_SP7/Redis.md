# Redis（RedisLogProvider）

> `EEPServerTools.Core/Adapter/Redis.cs` — 123 行

## 用途

**Redis 日誌與除錯緩衝提供者**（Redis Log & Debug Buffer Provider）。

RedisLogProvider 原本設計為透過 Redis 記錄 SQL 執行日誌和管理除錯用的二進位緩衝區。但目前 Redis 相關程式碼已全部被註解掉，改為使用記憶體內的靜態 Dictionary 作為替代實作。

此類別由 `Flow.CreateFlowApi` 建立，注入到 `EEP.Flow.API` 作為日誌和除錯提供者。

## 實作介面

| 介面 | 說明 |
|------|------|
| `EEP.Flow.ILogInfoProvider` | SQL 日誌記錄 |
| `EEP.Flow.IDebugProvider` | 除錯緩衝區管理（儲存/讀取/移除） |

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `RedisStr` | string | Redis 連線字串（目前未使用） |
| `LogKey` | string | 日誌鍵名（目前未使用） |
| `Enable` | bool | 是否啟用日誌，**固定回傳 `false`** |

## 核心方法

| 方法 | 說明 |
|------|------|
| `LogSql(commandText, dt, e)` | 記錄 SQL 執行日誌。目前為空實作（Redis 程式碼已註解） |
| `SaveBuffer(key, buffer)` | 將 byte[] 轉為 Base64 存入靜態 Dictionary（加鎖） |
| `LoadBuffer(key)` | 從靜態 Dictionary 讀取 Base64 字串並還原為 byte[]（加鎖） |
| `RemoveBuffer(key)` | 從靜態 Dictionary 移除指定 key（加鎖） |

## 關鍵邏輯

### 原始 Redis 設計（已註解）

- **LogSql**：將 SQL 文字、開始/結束時間戳（自 1970 epoch 毫秒）、錯誤訊息寫入 Redis List，設定 20 秒過期
- **SaveBuffer**：將 byte[] 存為 `debugger_{key}`，設定 30 分鐘過期
- **LoadBuffer / RemoveBuffer**：讀取/刪除 `debugger_{key}`
- **Enable**：透過 Redis 查詢 `enable{LogKey}` 是否存在來決定啟用狀態

### 目前實作

- 使用 `static Dictionary<string, string> Buffers` 替代 Redis
- 所有 Buffer 操作使用 `lock (typeof(RedisLogProvider))` 確保執行緒安全
- `LogSql` 完全為空（不記錄任何日誌）
- `Enable` 固定回傳 `false`

## 備註

- Redis 功能被停用的原因不明，可能是為了簡化部署或因效能考量。
- 使用 `lock (typeof(RedisLogProvider))` 鎖定型別物件是一種較粗粒度的鎖定方式，在高併發下可能影響效能。
- 靜態 Dictionary 沒有過期機制，長時間運行可能導致記憶體持續增長（除非 RemoveBuffer 被正確呼叫）。
- 此類別雖名為 Redis，但實際上目前不依賴 Redis，StackExchange.Redis 僅在 using 中引用。
