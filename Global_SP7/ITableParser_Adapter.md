# ITableParser (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/ITableParser.cs` |
| 行數 | 1526 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

將 AI 產出的 JSON（cards 結構）轉換為 ITable 設計器的 JSON 定義。此檔案同時包含 `RTableParser`（報表解析）、`HtmlParser`（JSON 轉 HTML 預覽）及相關資料類別。

## 主要類別

### ITableParser

- **輸入**：HTML（頁面名稱）、JSON（cards 陣列）、Setting（pageType / template / id）
- **輸出**：ITable JSON（controls 陣列 + iTableDesigner）

解析流程：
1. 從 cards 中辨識 Form / Grid / Query / ToolItem / None
2. 合併 Query 到 Master、合併 ToolItem
3. 根據 `pageType`（masterDetail）及 `template`（switch / open）調整控件結構
4. OnlyQuery 模式下全部 Readonly
5. MasterDetail 支援 Tab 分頁

### RTableParser（繼承 ITableParser）

- 專門處理報表定義，從 JSON 的 header / detail / footer 結構產生查詢 Form + 資料 Grid
- 支援 Excel 匯出模式：呼叫 `CreateExcelFile` 產生 NPOI Excel 並處理 `ExcelProvider.ProcessRFile`
- 支援群組標題（#GH）、群組小計（#GF）、報表頁尾（#RF）

### HtmlParser

- 將 JSON cards 轉為 Bootstrap HTML 預覽頁面
- 支援 Tab 分頁、Card、Form、Grid 等 HTML 結構

### ControlInfo / FieldInfo / TabInfo

| 類別 | 說明 |
|------|------|
| `ControlInfo` | 控件資訊（IsForm / IsGrid / IsQuery / IsToolItem），包含 ToJSON + Render |
| `FieldInfo` | 欄位資訊，自動辨識 FieldType（T/N/D/DT/A/C/IO/S/K/KA/KN） |
| `TabInfo` | Tab 分頁容器 |

### 欄位型別辨識

`FieldInfo` 使用 `TransformType.GetColumnType` + 中文名稱規則：
- 次序/項次/序號 → KA
- 依 TransformType 辨識（日期/金額/照片/檔案等）

## 備註

- `ITableParser.GetFieldType` 為靜態方法，供外部呼叫（如 FieldInfo）
- Excel 報表支援查詢欄位（#QT）、群組表頭/表尾、日期/列印者變數
- Key 欄位最多 2 個，超過自動降為 T
