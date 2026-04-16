# UserTableProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/UserTableProvider.cs` |
| 行數 | 294 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `UserDesignProvider` |

## 用途

提供資料表結構查詢與定義相關功能，用於設計端取得資料表 Schema、欄位名稱、欄位型別、表單定義及搬移定義等資訊。

## mode 分派

| mode | 說明 |
|------|------|
| `loadSchema` | 查詢資料表 Schema（欄位名、型別、允許 NULL、是否為 Key） |
| `getTableNames` | 取得所有表單名稱 |
| `getColumnNames` | 取得指定資料表的欄位名稱 |
| `getTableDefinations` | 取得多表定義（含 COLDEF、DOCFILES、XLSFILES 資訊） |
| `getColumnTypes` | 取得資料庫支援的欄位型別 |
| `getMoveDefinations` | 取得搬移定義（remoteName + coldef 欄位） |
| `getUserDict` | 以 `TransformType` 智慧辨識欄位型別 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetTableDefinations(tables)` | 依序查 SystemTable → SYS_DOCFILES → SYS_XLSFILES → COLDEF，組裝每個表的完整定義 |
| `SetTableDef(tableDef, coldefRows, captions)` | 設定 viewColumns / columns / valueTitle / textTitle |
| `GetMoveDefinations(tableName)` | 查 DOCFILES 或 XLSFILES 取得 remoteName 與欄位列表 |

## 備註

- `getTableDefinations` 優先查系統表（USERS/GROUPS/ROLES），接著查 Word 表單（SYS_DOCFILES）、Excel 表單（SYS_XLSFILES），最後 fallback 到 COLDEF
- `getUserDict` 使用 `TransformType.GetColumnType` 根據欄位名稱中文語義辨識型別
