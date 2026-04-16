# MenuGroupProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/MenuGroupProvider.cs` |
| 行數 | 82 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `MenuProvider` |

## 用途

選單群組管理 Provider，提供群組的 CRUD 及選單歸類功能。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 取得所有群組清單 |
| `add` | 新增群組 |
| `remove` | 移除群組 |
| `rename` | 重新命名群組 |
| `set` | 設定選單所屬群組（繼承自 MenuProvider） |

## 備註

- 群組資料透過 `GetMenuGroup()` / `SaveMenuGroup()` 讀寫（繼承自 MenuProvider）
- 群組結構為 `JObject`，key 為群組名稱，value 為 JArray
