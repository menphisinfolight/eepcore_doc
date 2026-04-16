# ColdefProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/ColdefProvider.cs` |
| 行數 | 147 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

管理欄位定義（COLDEF）資料表的設計時 Provider，提供 COLDEF 資料的查詢、新增、刪除與儲存功能。COLDEF 資料表儲存各資料表的欄位中文名稱、編輯器類型、驗證規則等元資料。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `load` | 載入 COLDEF 資料（透過 `DataModule.GetDataset` 查詢） |
| `add` | 為指定資料表自動建立 COLDEF 記錄（從 DB Schema 反推） |
| `remove` | 刪除指定資料表的所有 COLDEF 記錄 |
| `save` | 儲存 COLDEF 變更（透過 `DataModule.UpdateDataset`） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `LoadColdef(command, options)` | 載入 COLDEF 資料，支援分頁與篩選 |
| `AddColdef(table)` | 自動產生 COLDEF：1) 從 `dbHelper.GetTableSchema` 取得欄位結構 2) 篩選尚未有 title 的欄位 3) 查詢現有 COLDEF 嘗試比對 CAPTION 4) 產生 insertedRows 並批次寫入 |
| `RemoveColdef(table)` | 直接執行 `DELETE FROM COLDEF WHERE TABLE_NAME=...` |
| `SaveColdef(datas)` | 透過 `DataModule.UpdateDataset("coldef", datas)` 儲存 |

## 備註

- `AddColdef` 的邏輯：先取得資料表 Schema，再查詢系統中是否有其他表的同名欄位已有 CAPTION 定義（透過 `coldef` command），用於自動推測中文欄位名稱。
- `AddColdef` 對 datetime 型別的欄位自動設定 `NEEDBOX = "DateBox"`。
- `CHECK_NULL` 由 `allowNull` 反推：`allowNull == "1"` 設為 `"N"`，否則為 `"Y"`。
- `RemoveColdef` 使用 `dbHelper.MarkValue` 處理 TABLE_NAME 值的引號，避免 SQL 注入。
- `SEQ` 欄位從 1 開始遞增（僅針對尚未有 title 的欄位）。
