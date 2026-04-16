# TransProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/TransProvider.cs` |
| 行數 | 121 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `UserDesignProvider` |

## 用途

轉換定義（TRS）管理 Provider，提供載入、儲存及刪除轉換定義的功能。轉換定義儲存在系統資料表 `SYS_TRSFILES`。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 依 TRSNAME 載入轉換定義（info + tables + trans） |
| `save` | 儲存轉換定義（InsertRows） |
| `remove` | 刪除轉換定義 |

## 備註

- 轉換定義包含 info（NAME、TRSDATE、OWNER、MASTERTABLE）、tables（master/detail 對應表名）、trans（TRSFORMAT JSON）
- 儲存時以 `InsertRows` 寫入，相當於 upsert
- TABLE_NAMES 來自 `UserDesignProvider` 的常數定義
