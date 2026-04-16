# Parser (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/Parser.cs` |
| 行數 | 204 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

文件解析 Adapter，封裝 `ParserLibrary` 提供 Word / Excel 文件的解析（取得表單定義）及預覽（套版輸出）功能。

## 主要方法

| 方法 | 說明 |
|------|------|
| `ParseWord(filePath, newRule)` | 解析 Word 文件，回傳表單定義 JObject |
| `ParseExcel(filePath, newRule)` | 解析 Excel 文件，回傳表單定義 JObject |
| `PreviewWord(filePath, outputPath, masterRow, detailRows, flowRows, parameters)` | Word 套版預覽（支援 PDF / DOCX 輸出） |
| `PreviewLoopWord(filePath, outputPath, rows)` | Word 批次套版（多筆資料合併） |
| `PreviewExcel(filePath, outputPath, queryRow, detailRows, parameters)` | Excel 套版預覽（支援 PDF / XLSX 輸出） |
| `ExportSystemfile(outputPath, rows, fileType)` | 匯出系統文件（ModuleParser） |

## 備註

- 依賴 `ParserLibrary`（WordParser / ExcelParser / ModuleParser）
- 支援寫回（Writeback）功能、自訂日期/時間格式
- 預覽參數包含 User / UserName / Today / JSList / JCList / TRS 等
