# SchemaProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/SchemaProvider.cs` |
| 行數 | 532 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

提供資料庫結構查詢的設計時 Provider，用於設計工具中瀏覽資料表清單、欄位結構、資料預覽，以及 Server 元件（InfoCommand）的查詢功能。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `getTable` | 取得資料表清單或指定資料表的欄位結構 |
| `getColumn` | 取得欄位清單（可從 SQL 語句、資料表名稱或 remoteName 取得） |
| `viewData` | 預覽資料（執行 SQL 查詢並回傳分頁結果） |
| `getCommand` | 取得 Server 模組清單或指定模組下的 InfoCommand 清單 |
| `getDetailCommand` | 取得指定 Command 的明細 Command 清單 |
| `query` | 直接執行 SQL 語句（`ExecuteNonQuery`） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetTables(database, table, filter, withColdef)` | `table` 為空時回傳資料表清單（樹狀節點）；非空時回傳該表的欄位 Schema |
| `GetColumns(database, commandText, table, remoteName, used, autoParam)` | 取得欄位清單，優先從 `remoteName` 解析 InfoCommand 取得 SQL，再用 `dbHelper.GetSchema` 取得結構 |
| `ViewData(database, commandText, options, autoParam)` | 執行 SQL 查詢回傳 `DataTable.ToDynamic` 結果 |
| `GetCommands(moduleName)` | `moduleName` 為空時列出 `design/server/*.json` 檔案；非空時列出該模組的 InfoCommand |
| `GetDetailCommands(module, command)` | 回傳 `DataModule.GetDetailCommands` 結果 |
| `Query(database, commandText)` | 直接執行 SQL（非查詢用途） |

## 備註

- `database` 參數為空時預設使用 `ClientInfo.Database`。
- `GetColumns` 含有大量已註解的 SAP RFC 相關程式碼（`SapContent` 處理），支援 SAP 匯入/匯出參數與 Table 欄位的 Schema 解析。
- `GetTables` 的 `withColdef` 參數控制是否同時載入 coldef 定義。
- `autoParam` 參數傳遞至 `GetSchema`，控制是否自動處理 SQL 參數。
- `query` mode 直接執行 SQL，屬於高權限操作（設計時使用）。
