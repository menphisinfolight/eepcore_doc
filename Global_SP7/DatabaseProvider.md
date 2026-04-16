# DatabaseProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/DatabaseProvider.cs` |
| 行數 | 547 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

管理資料庫連線、結構與部署的設計時 Provider。提供資料庫清單管理、連線測試、建立資料庫/系統資料表、資料庫結構部署（跨資料庫同步）、SP/View 管理，以及結構比對匯出等功能。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `get` | 取得所有資料庫清單（標記目前選取的） |
| `set` | 切換目前工作資料庫（寫入 Session 與 Cookie） |
| `load` | 載入資料庫設定項目清單 |
| `save` | 儲存資料庫設定項目 |
| `test` | 測試資料庫連線 |
| `createDB` | 建立新資料庫 |
| `createSystemTable` | 在指定資料庫建立系統資料表 |
| `settings` | 取得資料庫設定（dbName、tableMarker、columnMarker、columnTypes） |
| `getSystemDatabase` | 取得系統資料庫名稱（目前回傳空字串） |
| `deploy` | 部署：依 `deployType` 分為資料表部署或選單部署 |
| `export` | 匯出選單（委託 `MenuProvider.DeployMenu`） |
| `sp` / `view` | 取得所有預存程序或檢視表清單 |
| `getView` | 取得指定 SP/View 的 SQL 內容 |
| `createViewSP` | 建立 SP/View |
| `deleteViewSP` | 刪除 SP/View |
| `saveViewSP` | 修改 SP/View（先 DROP 再 CREATE） |
| `compare` | 比對兩個資料庫結構差異並產生 `.sql` 檔案 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `InitKey()` | 初始化管理金鑰（產生隨機 2 bytes 寫入 `design/config/admin.txt`） |
| `CheckAndInit(key, datas)` | 驗證金鑰後建立系統資料表與儲存資料庫設定 |
| `SetDatabase(value)` | 切換資料庫並存入 Session 與 Cookie（7天有效） |
| `DeployDatabase(from, to, tables, includeData)` | 跨資料庫部署：比對結構差異並執行 SQL，可選擇同步資料 |
| `GetDeploySqlFile(from, to, tables, includeData)` | 產生部署 SQL 檔案（`.sql`）供下載 |
| `GetDeploySql(tableName, fromDB, toDB)` | 單一資料表的結構比對，產生 CREATE 或 ALTER SQL |
| `GetTableDataSql(tableName, fromDB, toDB)` | 產生資料同步 SQL（透過 `UpdateComponent` 產生 INSERT/UPDATE） |
| `GetAllSPView(mode)` | 取得所有 SP 或 View 清單 |
| `ChangeSPView(name, ctype, sql, param, changeType)` | 建立/刪除/修改 SP 或 View |

## 備註

- 管理金鑰存放於 `design/config/admin.txt`。
- 部署時 `deployType == "table"` 走資料表部署，其餘走 `MenuProvider.DeployMenu`。
- `export` mode 實際上也是呼叫 `MenuProvider.DeployMenu` 但 `isExport=true`。
- SP/View 的修改（alter）實作為先 DROP 再 CREATE。
- Oracle 的 procedure 會自動加括號包裹參數。
- `GetDatabaseItem` 方法為 public，可被其他 Provider 呼叫取得資料庫設定。
