# Parser

> `EEPServerTools.Core/Adapter/Parser.cs` — 340 行

## 用途

**報表解析與預覽介面卡**（Report Parser Adapter）。

Parser 負責解析 Word / Excel 範本檔案、產生報表預覽（docx / xlsx / pdf），以及匯出系統文件。依賴 `ParserLibrary`（WordParser、ExcelParser、ModuleParser）進行實際的文件處理。

## 實作介面

無實作介面。純功能類別，以 `ClientInfo` 初始化。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `ClientInfo` | ClientInfo | 當前使用者的連線資訊，提供 Locale 給解析器 |

## 核心方法

| 方法 | 說明 |
|------|------|
| `ParseWord(filePath, newRule)` | 解析 Word 範本，回傳文件結構（JObject）。驗證失敗時拋出 EEPException |
| `ParseExcel(filePath, newRule)` | 解析 Excel 範本，回傳文件結構（JObject）。驗證失敗時拋出 EEPException |
| `PreviewWord(filePath, outputPath, masterRow, detailRows, subDetailRows, flowRows, parameters)` | 產生 Word 報表預覽。支援輸出 docx 或 pdf 格式 |
| `PreviewLoopWord(filePath, outputPath, rows, parameters)` | 產生迴圈式 Word 報表（多筆資料合併為一份文件）。支援 docx / pdf |
| `PreviewExcel(filePath, outputPath, queryRow, detailRows, headRow, headTitle, parameters)` | 產生 Excel 報表預覽。支援 xlsx 或 pdf 格式 |
| `ExportSystemfile(outputPath, rows, fileType)` | 匯出系統文件（透過 ModuleParser） |

## 關鍵邏輯

### ParseWord / ParseExcel

```
讀取範本檔 → DisplayContent() 解析文件結構
  → DocResult() / ExcelResult() 取得結果物件
  → ValidateData() 驗證
  → 成功 → 序列化為 JObject 回傳
  → 失敗 → 拋出 EEPException（錯誤訊息）
```

### PreviewWord

1. 使用 `lock (_previewLock)` 確保同一時間只有一個預覽執行（靜態鎖）
2. 複製範本檔（加上 GUID 避免衝突）
3. 解析文件結構，合併主表資料（masterRow）到 MasterResultList 和 MasterRemoveList
4. 欄位名對應規則：先嘗試 `Name@ColumnName`，再嘗試 `Name`，最後以 ColumnName 查找
5. 處理 parameters（包含 Writeback、JSScript、JCScript、TRS 等擴充參數）
6. 依輸出格式產生檔案：
   - `.pdf` → 依 `printSetting` 設定選擇 Aspose 或內建方式轉 PDF
   - 其他 → `CreateDocFile()` 產生 docx
7. 刪除暫存的複製檔

### PreviewExcel

1. 解析 Excel 範本，合併查詢條件值（queryRow）到 EQueryList
2. 查詢欄位對應規則：`ColumnName@SearchMode` → `ColumnName` → `Name@SearchMode` → `Name`
3. 當 SearchMode 為 `%` 時，會依序嘗試 `%%`、`=`、`>`、`<` 等模式
4. 處理表頭（headRow）：Writeback 模式下，F SEQ 欄位輸出為 `#H{Type}` 格式
5. 支援 DateFormat、TimeFormat 自訂格式
6. 輸出 pdf 或 xlsx

### PreviewLoopWord

1. 解析文件結構並處理 parameters
2. 若輸出 pdf，先產生 docx 暫存檔再轉 pdf（Aspose 或內建）
3. 呼叫 `DeleteLoopFile()` 清理暫存

## 備註

- `EnableLog` 常數為 `false`，日誌功能預設關閉。
- PreviewWord 使用靜態鎖 `_previewLock`，這代表整個應用程式同一時間只能處理一個 Word 預覽請求，可能成為效能瓶頸。
- PDF 轉換支援兩種引擎：`Aspose`（由 `printSetting` 設定）和預設的 ParserLibrary 內建方式。
- Excel 的 PDF 轉換目前 Aspose 路徑被註解掉，僅使用 parser 內建方式。
- 欄位對應使用 `Name@ColumnName` 格式以處理同名欄位映射至不同資料庫欄位的情境。
