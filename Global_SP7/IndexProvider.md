# IndexProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/IndexProvider.cs` |
| 行數 | 63 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `BaseProvider` |

## 用途

資料庫索引管理 Provider，提供查詢、建立及刪除資料表索引的功能。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 查詢指定資料表的索引清單 |
| `create` | 建立索引（table + index name + column strings） |
| `drop` | 刪除索引 |

## 備註

- 操作對象為使用者資料庫（`DatabaseType.Normal`）
- SQL 語句由 `DatabaseHelper` 的 `GetCreateIndex` / `GetDropIndex` 產生
