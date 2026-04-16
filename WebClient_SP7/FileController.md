# FileController

> **檔案路徑**：`EEPWebClient.Core/Controllers/FileController.cs`（344 行）

## 用途

檔案上傳與下載的核心控制器，支援多種檔案類型（一般檔案、圖片、CSS、GeoJSON、SVG），並提供 EEP.NET 流程附件與聊天室圖片的專用存取端點。

## URL 路由

預設 MVC 路由：`/File/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | GET | `/File` | `f`（資料夾）、`q`（檔名）、`n`（顯示名稱） | FileResult / NotFound | 下載一般檔案；若帶 `n` 則視為匯出檔案 |
| `Index` | POST | `/File` | `IFormCollection`（含上傳檔案） | JSON / string | 上傳一般檔案 |
| `Images` | GET | `/File/Images` | `q`（檔名） | FileResult | 下載圖片 |
| `Images` | POST | `/File/Images` | `IFormCollection` | JSON / string | 上傳圖片 |
| `Css` | GET | `/File/Css` | `q`（檔名） | FileResult | 下載 CSS 檔 |
| `Css` | POST | `/File/Css` | `IFormCollection` | JSON / string | 上傳 CSS 檔 |
| `Geojson` | GET | `/File/Geojson` | `q`（檔名） | FileResult | 下載 GeoJSON 檔 |
| `Geojson` | POST | `/File/Geojson` | `IFormCollection` | JSON / string | 上傳 GeoJSON 檔 |
| `Svg` | GET | `/File/Svg` | `q`（檔名） | FileResult | 下載 SVG 檔 |
| `Svg` | POST | `/File/Svg` | `IFormCollection` | JSON / string | 上傳 SVG 檔 |
| `Umimage` | POST | `/File/Umimage` | `IFormCollection`、`f`（query） | JSON string | UEditor 圖片上傳，回傳 `{url, state}` |
| `Flow` | GET | `/File/Flow` | `f`、`q`、`n` | FileResult | 流程附件下載（自動判斷 EEP.NET 或本地） |
| `Flow` | POST | `/File/Flow` | `IFormCollection` | JSON | 流程附件上傳 |
| `NetFlow` | GET | `/File/NetFlow` | `f`、`q`、`n` | FileResult | EEP.NET 流程附件下載 |
| `NetFlow` | POST | `/File/NetFlow` | `IFormCollection` | JSON `[{name, size}]` | EEP.NET 流程附件上傳 |
| `ChatImage` | GET | `/File/ChatImage` | `q`（檔名） | FileResult | 聊天室圖片下載 |
| `ChatImage` | POST | `/File/ChatImage` | `IFormCollection`（含 `id`） | JSON `[{name, text, size}]` | 聊天室圖片上傳 |

## 關鍵邏輯

### 檔案下載（`GetFile`）

1. 透過 `AccountProvider` 從 Referer 取得 `clientInfo`（未登入則回傳空）
2. 由 `FileProvider.GetFilePath()` 依 `FileType` 與資料夾組合實體路徑
3. 支援 `?t=inline` 查詢參數切換 Content-Disposition 為 inline（瀏覽器內顯示）或 attachment（下載）
4. 使用 `filename*=UTF-8''` 格式的 Content-Disposition 處理中文檔名
5. 針對 `.docx`、`.xlsx`、`.pdf`、`.txt` 設定專屬 Content-Type

### 檔案上傳（`UploadFile`）

1. 驗證 Session 登入狀態
2. 委派 `FileProvider.UpLoadFile()` 處理實際存檔
3. 回傳結果可能為 string 或 JSON 物件

### Flow 分流

- `Flow` action 透過 `EEPNetHelper.IsEEPNETFlow` 判斷是否為 EEP.NET 環境
- EEP.NET 模式下由 `EEPNetFlowProvider` 處理上傳/下載

### ChatImage 存檔

- 上傳路徑：`design/chat/{Solution}/{id}_{timestamp}{ext}`
- 檔名格式含時間戳防止覆蓋

## 備註

- `FileType` 列舉值包含：`Files`、`Images`、`Css`、`Geojson`、`Svg`、`Export`、`Doc` 等
- `Umimage` 為 UEditor 富文本編輯器的圖片上傳接口，回傳格式符合 UEditor 規範
- `GetFile` 中的 `PhysicalFileProvider` 以 `Environment.CurrentDirectory` 為根目錄
