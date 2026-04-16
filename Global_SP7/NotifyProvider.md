# NotifyProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/NotifyProvider.cs` |
| 行數 | 35 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

通知管理 Provider（目前為空實作）。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 取得通知清單（目前回傳空陣列 `[]`） |
| `remove` | 刪除通知（目前回傳空字串） |

## 備註

- 此 Provider 目前為預留介面，`GetNotifies` 和 `DeleteNotify` 均為空實作
