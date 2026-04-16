# ImagesController

> **檔案路徑**：`EEPWebClient.Core/Controllers/ImagesController.cs`（39 行）

## 用途

提供系統品牌圖片（Logo、登入 Logo、背景圖）的存取端點，支援方案層覆蓋（優先讀取方案圖片，不存在時回退到系統預設）。

## URL 路由

預設 MVC 路由：`/Images/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Login_logo` | GET | `/Images/Login_logo` | 無 | FileResult | 登入頁 Logo（`login_logo.png`） |
| `Logo` | GET | `/Images/Logo` | 無 | FileResult | 系統 Logo（`logo.png`） |
| `Bg_main` | GET | `/Images/Bg_main` | 無 | FileResult | 主背景圖（`bg_main.png`） |

## 關鍵邏輯

### 圖片讀取（`GetImage`）

1. 透過 `FileProvider.GetFilePath(FileType.Images, "", fileName)` 取得方案層圖片路徑
2. 若方案層圖片不存在，回退至 `FileProvider.GetSystemFilePath(FileType.Images, fileName)` 取得系統預設圖片
3. 讀取整個檔案為 byte[] 後以 `application/octet-stream` 回傳

## 備註

- 不需登入即可存取（未檢查 Session）
- 圖片 fallback 機制允許各方案自訂品牌，未自訂時使用系統預設
- Content-Type 統一使用 `application/octet-stream`，瀏覽器會依副檔名自動判斷
