# StyleSheetsController

> **檔案路徑**：`EEPWebClient.Core/Controllers/StyleSheetsController.cs`（34 行）

## 用途

動態產生 CSS 樣式表，提供主樣式與全域樣式的端點，以 `text/css` 格式回傳給瀏覽器。

## URL 路由

預設 MVC 路由：`/StyleSheets/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Main` | GET | `/StyleSheets/Main` | 無 | CSS 檔案 | 主要樣式表（`index.css`） |
| `Global` | GET | `/StyleSheets/Global` | 無 | CSS 檔案 | 全域樣式表（`global.css`） |

## 關鍵邏輯

1. 從 Session 取得 `ClientInfo`，未登入回傳空內容
2. 由 `GlobalFileProvider.GetGlobalFile("index"/"global", "css")` 讀取對應 CSS 檔案
3. 檔案來源依方案設定

## 備註

- 結構與 `ScriptsController` 的 `Main`/`Global` 完全對稱，差異僅在副檔名（`css` vs `js`）與 Content-Type（`text/css;charset=utf-8`）
- 需登入才能取得樣式（未登入回傳空）
