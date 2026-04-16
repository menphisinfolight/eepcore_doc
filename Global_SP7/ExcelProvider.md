# ExcelProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/ExcelProvider.cs` |
| 行數 | 846 |
| 繼承 | `UserDesignProvider` > `BaseProvider` |

## 用途

管理 Excel 樣板文件（`.xlsx`）的設計時操作，包括上傳解析、載入、儲存、移除與回寫。與 `WordProvider` 類似但針對 Excel 報表樣板。解析後的定義儲存至 `SYS_XLSFILES` 系統資料表，支援列印欄位（Print）、查詢欄位（Query）與表頭欄位（Head）三種列定義。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `load` | 載入已儲存的 Excel 定義（從 `SYS_XLSFILES` 讀取） |
| `save` | 儲存 Excel 定義至 `SYS_XLSFILES`（含 REPORTFORMAT、QUERYFORMAT、HEADFORMAT） |
| `remove` | 刪除 Excel 定義 |
| `write` | 回寫 Excel 文件：將欄位定義寫回 `.xlsx` 檔 |

## ProcessFile

處理 Excel 檔案上傳，呼叫 `AddExcel` 進行解析：
1. 儲存上傳的 `.xlsx` 至 `design/doc/{Solution}/`
2. 透過 `Parser.ParseExcel` 解析 Excel 內容
3. 依資料表是否已有 Word 定義或其他 Excel 定義，決定欄位來源
4. 產生 PrintRows、QueryRows 並組合 coldefs

## ProcessRFile

供外部呼叫的方法，直接以 `clientInfo` 和 `remoteName` 處理指定 Excel 檔案（不經 HTTP 請求），解析後自動儲存。

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `AddExcel(id, path, newRule)` | 解析 Excel 並產生 excelData（info、coldefs、codes、setting） |
| `LoadExcel(id)` | 從 `SYS_XLSFILES` 載入定義，同時查詢關聯的 Word 定義取得 DocName |
| `SaveExcel(id, datas)` | 儲存定義至 `SYS_XLSFILES`，同步處理 sysParas |
| `RemoveExcel(id)` | 刪除 `SYS_XLSFILES` 記錄 |
| `WriteExcel(id, datas, exportFile)` | 呼叫 `Parser.PreviewExcel` 回寫檔案 |
| `ToPrintRows(items, coldefs, ...)` | 將解析結果轉為列印欄位定義（含型別辨識與 coldef 比對） |
| `ToPrintRow(item, coldef, confirm)` | 單筆列印欄位轉換，處理型別繼承邏輯 |
| `ToQueryRows / ToQueryRow` | 將解析結果轉為查詢欄位定義，支援多來源比對（coldef、printRow、recogRow） |
| `ToExcelRow / ToExcelValue` | 將 coldef 轉回 Excel 標記格式（如 `#T`、`#Q`、`##N` 等） |

## 備註

- 常數路徑與 `WordProvider` 一致：`FILE_ROOT`、`DOC_ROOT`、`EXPORT_ROOT`。
- 支援 `EXCEL_EDIT` 型別（表示此 Excel 為編輯用途而非僅列印）。
- 載入時會查詢 `SYS_DOCFILES` 尋找關聯的 Word 定義，以取得 `DocName` 和明細表名稱。
- 查詢欄位支援多種搜尋模式（`=`、`%`、`>`、`<`、`>=`、`<=`）。
- `confirm` 標記用於首次建立時提示使用者確認（`'{tableName}':wordFirst`）。
- `ProcessRFile` 方法允許程式碼直接呼叫（不經 HTTP），用於自動化處理。
