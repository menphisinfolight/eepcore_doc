# ParserHelper

> `EEPServerTools.Core/Utility/ParserHelper.cs` — 969 行

## 用途

**報表匯出輔助工具**（Report Parser Helper）。

ParserHelper 是 EEP Core 伺服器端的報表匯出類別，負責將資料查詢結果匯出為 Word（.docx）或 Excel（.xlsx）格式的報表。核心功能包括：從系統資料表取得欄位定義（coldef）與報表格式設定、進行欄位值對應（value → text）、格式化數值/日期/圖片欄位，最終透過 `Parser` Adapter 產生實際檔案。

同時包含 `ExportWordOptions` 和 `ExportExcelOptions` 兩個選項類別。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Context` | HttpContext | HTTP 請求上下文 |
| `ClientInfo` | ClientInfo | 當前使用者的連線資訊 |
| `DocDir` | string | 報表範本目錄 |
| `ExportDir` | string | 匯出檔案輸出目錄 |
| `FileDir` | string | 一般檔案目錄（圖片等） |
| `FlowDir` | string | 流程相關檔案目錄 |

## 核心方法

| 方法 | 說明 |
|------|------|
| `ExportWord(string id, ExportWordOptions)` | 匯出單筆 Word/PDF 報表 |
| `ExportWordLoop(string id, ExportWordOptions, QueryOptions)` | 批次迴圈匯出 Word（多筆資料合併為單一檔案） |
| `ExportWordAll(string id, ExportWordOptions, QueryOptions)` | 批次匯出多個 Word 檔案並打包為 ZIP |
| `ExportExcel(string id, ExportExcelOptions)` | 匯出 Excel/PDF 報表 |

## Word 匯出流程

### ExportWord（單筆匯出）

```
ExportWord(id, options)
  → GetWordName(id)          // 從 SYS_DOCFILES 取得範本檔名
  → 定位範本路徑              // 優先 {DocDir}/{Solution}/{wordName}，fallback 到 {DocDir}/{wordName}
  → ExportsWord(filePath, options, masterRow)
```

### ExportsWord（核心匯出邏輯）

```
ExportsWord(filePath, options, masterRow)
  ├── 1. 取得流程簽核紀錄（若有 FlowFlag）
  │   └── GetFlowRows(instanceID) → 查詢流程歷史，組裝 WN/WC/WU/WR/WT/WS/WD 欄位
  │
  ├── 2. 取得欄位定義
  │   └── GetWordColdef(moduleName, commandName, rows) → 查 coldef 資料表
  │
  ├── 3. 組裝檔名
  │   └── 時間戳記 + 主鍵值（IS_KEY=Y 的欄位）+ .docx/.pdf
  │
  ├── 4. 取得明細資料
  │   └── GetDetailCommands → 逐一查詢明細資料 → FormatRow 格式化
  │
  ├── 5. 格式化主表資料
  │   └── FormatRow(masterRow, coldefs)
  │
  └── 6. 呼叫 Parser 產生檔案
      ├── .xlsx 範本 → Parser.PreviewExcel()
      └── .docx 範本 → Parser.PreviewWord()
```

### ExportWordLoop（批次合併匯出）

查詢多筆主表資料，每筆各自取得明細和流程紀錄，最後一次呼叫 `Parser.PreviewLoopWord` 產生合併檔案。

### ExportWordAll（批次獨立匯出 + ZIP）

查詢多筆主表資料，每筆各自呼叫 `ExportsWord` 產生獨立 Word 檔，最後將所有檔案打包為 ZIP 回傳。

## Excel 匯出流程

### ExportExcel

```
ExportExcel(id, options)
  ├── 1. 定位範本路徑（{DocDir}/{Solution}/{id}.xlsx 或 excelName）
  │
  ├── 2. 取得欄位格式設定
  │   └── GetExcelColRows(id) → 從 SYS_XLSFILES 取得 queryColRows 和 detailColRows
  │
  ├── 3. 查詢明細資料
  │   ├── 依 detailColRows 中 G/GR 類型欄位排序
  │   ├── 支援 WhereItems / WhereStr 篩選
  │   └── 若有 FIELD_DETAILEDCOLUMN=Y → 額外查詢子明細（PrintDetailList）
  │
  ├── 4. 組裝查詢列（queryRow）
  │   └── 從 WhereItems 擷取欄位與值，處理比較運算符
  │
  ├── 5. 格式化欄位
  │   └── GetExcelColdef → FormatRow 處理查詢列與明細列
  │
  └── 6. 呼叫 Parser.PreviewExcel 產生檔案
```

## 欄位定義與格式化

### 欄位類型（NEEDBOX）

| 類型碼 | 說明 | 格式化行為 |
|--------|------|------------|
| `N` | 數值 | 依 DESCRIPTION 格式化（`N`=去尾零、其他=ToString(format)） |
| `D` | 日期 | 轉為 `yyyy-MM-dd` |
| `DC` | 緊湊日期（yyyyMMdd） | 解析後轉為 `yyyy-MM-dd` |
| `DT` | 日期時間 | 轉為 `yyyy-MM-dd HH:mm:ss` |
| `P` | 圖片 | 組裝圖片完整路徑（FileDir/Images/value） |
| `R` / `C` / `GR` / `QR` | 關聯欄位 | 查詢關聯表取得 value→text 對應 |
| `G` / `GR` | 群組排序欄位 | 用於 Excel 匯出時的排序依據 |
| `GF` / `GH` | 群組頁首/頁尾 | 以 CAPTION 作為 key（非 FIELD_NAME） |

### 值對應（valueTexts）

關聯欄位（R/C/GR/QR）會依 DESCRIPTION 指定的表名查詢對應文字：

| 來源表 | 對應規則 |
|--------|----------|
| `roles` / `系統角色表` / `系统角色表` / `groups` / `系統部門表` / `系统部门表` | GROUPID → GROUPNAME |
| `users` / `系統用戶表` / `系统用户表` | USERID → USERNAME |
| 其他自訂表 | 從 SYS_DOCFILES 查 KEYFIELDS / NAMEFIELDS |

### FormatRow

將原始資料列依欄位定義進行：
1. 欄位名稱替換為 CAPTION（中文標題）
2. 值對應（valueTexts：代碼 → 顯示文字）
3. 數值/日期/圖片格式化
4. 遞迴處理 `PrintDetailList`（子明細）

## 流程簽核欄位

Word 報表中的流程簽核區塊使用固定欄位名：

| 欄位 | 說明 |
|------|------|
| `WN` | 關卡名稱（ActivityText） |
| `WC` | 簽核意見（Remark） |
| `WU` | 簽核人（UserName） |
| `WR` | 角色名稱（RoleName） |
| `WT` | 狀態文字（如「核准」「退回」） |
| `WT2` | 狀態碼（原始數值） |
| `WS` | 電子簽章圖片路徑（design/signature/{UserID}.png） |
| `WD` | 簽核日期時間（Datetime） |

## 選項類別

### ExportWordOptions

| 屬性 | 類型 | 說明 |
|------|------|------|
| `RemoteName` | string | 資料來源（格式：`moduleName.commandName`） |
| `WordName` | string | 指定範本檔名（覆蓋 SYS_DOCFILES 設定） |
| `FileName` | string | 指定輸出檔名 |
| `MasterRow` | JObject | 主表資料（若為 null 則自動查詢） |
| `FileType` | string | 輸出格式：`pdf` 或 `docx`（預設） |
| `PageSize` | int | 批次匯出每頁筆數 |
| `WhereItems` | JArray | 篩選條件 |
| `WhereStr` | string | 額外 WHERE 字串 |
| `OutputPath` | string | 自訂輸出路徑 |
| `Password` | string | PDF 密碼保護 |
| `Watermark` | string | 浮水印文字 |

### ExportExcelOptions

| 屬性 | 類型 | 說明 |
|------|------|------|
| `RemoteName` | string | 資料來源（格式：`moduleName.commandName`） |
| `WhereItems` | JArray | 篩選條件 |
| `WhereStr` | string | 額外 WHERE 字串 |
| `PageSize` | int | 查詢筆數限制 |
| `DetailRows` | JArray | 直接傳入明細資料（不查詢） |
| `FileType` | string | 輸出格式：`pdf` 或 `xlsx`（預設） |
| `ExcelName` | string | 指定範本檔名 |
| `HeadTitle` | string | 表頭標題 |
| `Password` | string | PDF 密碼保護 |
| `Watermark` | string | 浮水印文字 |

## 備註

- 範本檔案路徑優先搜尋 `{DocDir}/{Solution}/` 下，找不到則 fallback 到 `{DocDir}/`。
- Word 範本支援 `.xlsx` 副檔名：若範本為 Excel 格式，會改用 `Parser.PreviewExcel` 產生。
- `ExportWordAll` 使用暫存資料夾 + `ZipFile` 打包，完成後刪除暫存。
- 流程簽核紀錄中 Status=3（抽回）的記錄會被排除，不顯示在報表中。
- Excel 匯出預設最大筆數為 1000（`queryOptions.PageSize = 1000`）。
- `GetWordColdef` 和 `GetExcelColRows` 分別從 `coldef`（SYS_DOCFILES 相關）和 `SYS_XLSFILES` 取得欄位格式設定。
- 支援 PDF 輸出（密碼保護、浮水印），由 `Parser` Adapter 實作轉換。
- **`ExportWord`（L329-480）有冗餘/未完成的 subDetailCommands 邏輯**：
  - L411、L415：`subDetailCommands` 宣告在 `if/else` block 內，變數作用域受限於該區塊，**純死碼**（宣告後即失效）。
  - L432：`GetDetailCommands(commandName)` 的呼叫參數與 L407 完全相同，取得的是同一組 detail commands；後續 foreach 重複查詢 detail 資料並填入 `sdRows`。`sdRows` 雖有傳給 `Parser.PreviewWord`，但內容與 `dRows` 重複。推測原意是查 sub-details（明細的明細），但實作未完成。
- **`ExportWordLoop`（L482-571）沒有此問題**，只有一個乾淨的 `detailCommands` 迴圈（L542-554），沒用到 `subDetailCommands`。
