# FileProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/FileProvider.cs` |
| 行數 | 1641 |
| 繼承 | `MenuProvider` > `DesignProvider` > `BaseProvider` |

## 用途

處理檔案匯出匯入的核心 Provider，涵蓋 Excel/Word/PDF/Report 匯出、Excel 匯入、檔案上傳、Logo 設定，以及表格式資料匯出等功能。同時包含報表 JSON 動態產生邏輯（PDF 報表版面配置）。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `exportDataset` | 透過 `DataModule` 將資料集匯出為 `.xlsx` |
| `exportFile` | 依 type 參數分派匯出：`word`、`wordLoop`、`wordAll`、`excel`、`report`、`pdf` |
| `importexcel` | 匯入 Excel 檔（上傳後透過 `DataModule.ImportXlsx` 匯入資料庫） |
| `import` | 匯入檔案，依 `menuType` 分為匯入 Schema（table）或匯入選單（menu） |
| `setLogo` | 上傳 Logo 圖片至 `Images` 資料夾 |
| `exportTable` | 將前端傳來的 JArray 表格資料匯出為 Excel（含合併儲存格） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `ExportToExcel(datas, columns, title)` | 將 JArray 資料依 columns 定義寫入 NPOI Workbook 並輸出 `.xlsx` |
| `ExportTableToExcel(datas, title)` | 處理含 rowspan/colspan 合併儲存格的表格匯出 |
| `ExportXlsx(module, command, ...)` | 透過 `DataModule.ExportXlsx` 匯出資料集 |
| `UpLoadFile(param, fileType)` | 通用檔案上傳，支援自動編號、副檔名過濾 |
| `ImportXlsx(param)` | 上傳 Excel 後呼叫 `DataModule.ImportXlsx` 匯入 |
| `ImportFile(param)` | 匯入 JSON/ZIP 檔案，分為 Schema 匯入或 Menu 匯入 |
| `ExportWord / ExportWordLoop / ExportWordAll` | 委託 `ParserHelper` 產生 Word 匯出檔 |
| `ExportExcel(id, options)` | 委託 `ParserHelper` 產生 Excel 匯出檔 |
| `ExportPdf(id, param)` | 讀取方案 JSON 設計檔，組合欄位後呼叫 `ReportPdf` 匯出 PDF |
| `ExportReport(id, options)` | 直接呼叫 `ReportPdf.exportToReport` 產生報表 |
| `GetReportJson(param)` | 根據參數動態產生報表 JSON 定義（頁首/頁尾/主體/明細/連結區段），自動計算頁面大小與版面配置 |
| `SetCellValueSafe(row, colIndex, value)` | 靜態方法，安全寫入 NPOI Cell（處理 null、超長文字截斷、各種數值型別） |
| `SanitizeExcelText(s)` | 清理 Excel 文字內容，移除控制字元並截斷至 32767 字元上限 |

## 備註

- 內含輔助類別 `RowMerge`，用於追蹤 rowspan/colspan 偏移量計算。
- `UpLoadFile` 標記 `[DisableRequestSizeLimit]`，允許大檔案上傳。
- `ExportPdf` 從 `design/bootstrap/{Solution}/{id}.json` 讀取設計定義檔。
- `GetReportJson` 會依欄位數量自動選擇紙張大小（A4/A3）與橫直向。
- 匯出過程均有 `LogHelper` 記錄操作日誌與耗時。
