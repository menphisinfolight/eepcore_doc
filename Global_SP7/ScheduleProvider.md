# ScheduleProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/ScheduleProvider.cs` |
| 行數 | 94 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

排程管理 Provider，提供排程任務的載入、儲存及排程日誌的查詢與清除。資料儲存在系統資料表 `schedule` / `SYS_SCHEDULE_LOG`。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 載入當前 database + solution 的排程清單 |
| `logs` | 查詢指定排程的執行日誌（支援分頁） |
| `clearLogs` | 清除指定排程的所有日誌 |
| `save` | 儲存排程（新增時自動帶入 DBAlias + Solution） |

## 備註

- 排程資料以 `DatabaseType.System` 存取系統資料庫
- 新增排程時自動設定 `DBAlias` 和 `Solution` 欄位
