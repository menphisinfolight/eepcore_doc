# WordProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/WordProvider.cs` |
| 行數 | 697 |
| 繼承 | `UserDesignProvider` > `BaseProvider` |

## 用途

管理 Word 樣板文件（`.docx`）的設計時操作，包括上傳解析、載入、儲存、移除與回寫。上傳的 Word 文件經 `Parser` 解析後，產生欄位定義（coldef）並儲存至 `SYS_DOCFILES` 系統資料表。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `load` | 載入已儲存的 Word 定義（從 `SYS_DOCFILES` 讀取欄位與設定） |
| `getColumnNames` | 取得指定資料表的欄位名稱 |
| `save` | 儲存 Word 定義（含 coldef、setting、codes）至 `SYS_DOCFILES` |
| `remove` | 刪除 Word 定義及相關 coldef 資料 |
| `write` | 回寫 Word 文件：將欄位定義寫回 `.docx` 檔（可選擇匯出或覆蓋原檔） |

## ProcessFile

處理 Word 檔案上傳，呼叫 `AddWord` 進行解析：
1. 儲存上傳的 `.docx` 至 `design/doc/{Solution}/`
2. 透過 `Parser.ParseWord` 解析 Word 內容
3. 透過 `TransformType.GetType` 進行型別轉換
4. 組合 coldef 列（`ToColdefRows`）、處理欄位比對與辨識

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `AddWord(id, path, newRule)` | 解析 Word 檔並產生完整的 wordData（info、coldefs、codes、setting） |
| `LoadWord(id)` | 從 `SYS_DOCFILES` 載入 Word 定義，組合各 TABLE_NAME 的 coldef |
| `SaveWord(id, datas)` | 將編輯後的定義存回 `SYS_DOCFILES`，同步儲存 sysParas |
| `RemoveWord(id)` | 刪除 `SYS_DOCFILES` 記錄及相關 tableNames |
| `WriteWord(id, datas, exportFile)` | 將 coldef 轉為 Word 標記後呼叫 `Parser.PreviewWord` 回寫檔案 |
| `ToColdefRows(name, items, coldefs, rules)` | 將解析結果與現有 coldef 比對合併，產生最終欄位定義列 |
| `ToColdefRow(item, coldefs, length)` | 單筆欄位轉 coldef row，處理型別繼承、欄位可見性等邏輯 |
| `ToGridColumn(childInfo)` | 處理 J 型別（grid column）的子欄位定義 |
| `toWordRow / toWordValue` | 將 coldef 轉回 Word 標記格式字串（如 `#T`、`#R`、`##N` 等） |

## 備註

- 常數路徑：`FILE_ROOT = design/files`、`DOC_ROOT = design/doc`、`EXPORT_ROOT = design/export`。
- 支援 Master / Detail / SubDetail 三層結構（`columnMark` 分別為 `#`、`##`、`###`）。
- `GetDescriptionRows` 會查詢 `COLDEF_DESCRIPTION` 表取得欄位說明輔助辨識。
- `newRule` 參數控制是否備份原始檔案及使用新解析規則。
- 上傳時若已有同名定義會保留既有 `REPORTFORMAT` 設定。
