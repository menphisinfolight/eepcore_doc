# UserDesignController

> **檔案路徑**：`EEPWebClient.Core/Controllers/UserDesignController.cs`（181 行）

## 用途

使用者自訂報表設計功能，處理 Word/Excel 報表範本的匯入、上傳、處理與下載。僅限開發模式（`Dev`）使用者存取。

## URL 路由

- 預設 MVC 路由：`/UserDesign/{action}`
- 動態路由：`/userdesign/{fileType}/{fileName}` 及 `/userdesign/{fileType}/backup/{fileName}`（由 `DownloadFileRouteTransformer` 轉換）

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | POST | `/UserDesign` | `IFormCollection`（含 `type`） | JSON / string | 依 `type` 參數建立對應 Provider 處理設計請求 |
| `Add` | POST | `/UserDesign/Add` | `IFormCollection`（含上傳檔案） | JSON / string | 上傳 Word/Excel 報表範本並處理 |
| `Upload` | POST | `/UserDesign/Upload` | `IFormCollection`（含上傳檔案） | JSON `[{name, size}]` | 上傳 Word/Excel 原始檔案（僅存檔，不處理） |
| `DownloadFile` | GET | `/UserDesign/DownloadFile` | `fileType`、`fileName` | FileResult | 下載報表範本檔案 |

## 關鍵邏輯

### 權限檢查

所有 Action 皆要求 `clientInfo.Dev == true`（開發模式），否則拋出 `EEPException("Timeout")`。`DownloadFile` 例外，僅檢查登入狀態。

### Index（設計請求處理）

- 由 `UserDesignProvider.CreateProvider(HttpContext, param["type"])` 建立對應型別的 Provider
- 呼叫 `provider.ProcessRequest(param)` 處理設計邏輯

### Add（報表範本匯入）

1. 從上傳檔案的副檔名判斷類型：`.doc/.docx` → `word`、`.xls/.xlsx` → `excel`
2. 建立對應 `UserDesignProvider` 並呼叫 `ProcessFile(param)` 解析範本
3. 不支援的副檔名拋出例外

### Upload（檔案上傳）

1. 同樣依副檔名判斷類型，建立 Provider
2. 透過 `provider.GetFilePath()` 取得目標路徑
3. 使用 `FileStream` 直接寫入檔案

### DownloadFile（檔案下載）

1. 透過 `FileProvider.GetFilePath(FileType.Doc, solution, fileName)` 取得檔案路徑
2. 若路徑包含 `backup` 且檔案不存在，嘗試移除 `backup\` 回退到原始路徑
3. 若方案層路徑不存在，改嘗試系統層路徑（`folder` 為空）
4. 備份檔案下載時自動在檔名後加上 `_下載` 後綴

## 備註

- 支援的檔案類型僅限 Word（`.doc`/`.docx`）與 Excel（`.xls`/`.xlsx`）
- `DownloadFile` 的動態路由由 `DownloadFileRouteTransformer` 提供，支援 `userdesign/{fileType}/{fileName}` 與 `userdesign/{fileType}/backup/{fileName}` 兩種模式
- `Add` 與 `Upload` 的差異：`Add` 會呼叫 `ProcessFile` 解析範本內容，`Upload` 僅存檔
