# TableProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/TableProvider.cs` |
| 行數 | 400 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

管理資料庫資料表的設計時 Provider，提供資料表內容瀏覽/編輯、結構（Schema）查看/修改、資料匯出為 Excel、結構匯出為 `.sqlx`、資料全部刪除、批次更新等功能。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `load` | 載入資料表內容（分頁查詢） |
| `save` | 儲存資料表變更（透過動態建立 InfoCommand + UpdateComponent） |
| `loadDataTypes` | 取得資料庫支援的資料型別清單 |
| `loadSchema` | 載入資料表結構（欄位名稱、資料型別、允許 NULL、主鍵） |
| `changeSchema` | 修改資料表結構（新增/修改/刪除欄位） |
| `exportexcel` | 將資料表內容匯出為 Excel（`.xlsx`） |
| `deleteAll` | 刪除資料表所有資料 |
| `export` | 匯出資料表結構為 `.sqlx` 檔案（JSON 格式，可選包含資料） |
| `getColumns` | 取得指定資料表的欄位清單（FIELD_NAME + CAPTION） |
| `importSampleOrg` | 匯入範例組織資料 |
| `batchUpdate` | 批次更新：將指定欄位設為另一欄位值或固定值 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `LoadTable(tableName, options, keys)` | 使用 `dbHelper.GetDataTable` 分頁載入資料 |
| `SaveTable(rows)` | 動態建立 `InfoCommand` + `UpdateComponent`，透過 `DataModule.UpdateDataset` 執行儲存 |
| `LoadDataTypes()` | 回傳 `dbHelper.GetDataTypes()` 結果 |
| `LoadSchema(tableName)` | 回傳資料表結構（name、dataType、allowNull、key） |
| `ChangeSchema(tableName, columns, keys, changeType)` | 呼叫 `dbHelper.ChangeSchema` 修改結構 |
| `ExportExcel(tableName, options)` | 使用 NPOI 將查詢結果匯出為 Excel，支援 coldef 取得欄位標題，整數/小數使用數值格式 |
| `ExportSchema(tableNames, includeData)` | 將多個資料表結構（含選擇性資料）匯出為 `.sqlx` JSON 檔 |
| `DeleteAll(tableName)` | 呼叫 `dbHelper.GetDeleteAllSql` 取得刪除 SQL 並執行 |
| `GetColumnsByTableName(tableName)` | 從 Schema 取得欄位清單（FIELD_NAME + 標題） |
| `ImportSampleOrg()` | 呼叫 `dbHelper.ImportSqlData("sampleOrg")` 匯入範例資料 |

## 備註

- `ExportExcel` 會查詢 `coldefForWord` 取得欄位中文標題作為 Excel 標頭。
- `ExportExcel` 的 `tableName` 若包含 `select` 關鍵字會直接作為 SQL 執行。
- `SaveTable` 採用動態元件方式，不依賴預先定義的 Server JSON，而是即時建立 InfoCommand。
- `batchUpdate` mode 直接拼接 SQL（`valueType` 為 `field` 時不加引號，為 `value` 時加 `N'...'`），注意無防注入處理。
- `.sqlx` 匯出格式為 JArray 的 JSON，每筆包含 `tableName`、`tableSchema`、`insertStrings`。
- `ColumnStyles` 字典用於 Excel 匯出時快取欄位的 `ICellStyle`，避免重複建立。
