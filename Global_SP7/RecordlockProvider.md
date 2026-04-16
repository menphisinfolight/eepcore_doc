# RecordlockProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/RecordlockProvider.cs` |
| 行數 | 49 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

記錄鎖定管理 Provider，提供查詢與釋放記錄鎖定的功能。支援 database 模式（`DBLockHelper`）和記憶體模式（`LockHelper`）兩種。

## mode 分派

| mode | 說明 |
|------|------|
| `get` | 查詢鎖定記錄（支援條件篩選） |
| `release` | 釋放指定鎖定 |

## 備註

- 鎖定模式由 `global.cfg` 的 `recordLock` 設定決定：`"database"` 用 `DBLockHelper`，其他用 `LockHelper`
