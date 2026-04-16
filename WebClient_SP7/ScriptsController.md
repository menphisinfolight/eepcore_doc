# ScriptsController

> **檔案路徑**：`EEPWebClient.Core/Controllers/ScriptsController.cs`（69 行）

## 用途

動態產生 JavaScript 腳本，包含多語系訊息、全域腳本、地圖腳本與自訂腳本，以 `application/javascript` 格式回傳給瀏覽器。

## URL 路由

預設 MVC 路由：`/Scripts/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Locale` | GET | `/Scripts/Locale` | 無 | JS 檔案 | 多語系訊息腳本（依使用者語系） |
| `Main` | GET | `/Scripts/Main` | 無 | JS 檔案 | 主要全域腳本（`index.js`） |
| `Global` | GET | `/Scripts/Global` | 無 | JS 檔案 | 全域腳本（`global.js`） |
| `Map` | GET | `/Scripts/Map` | 無 | JS 檔案 | 地圖相關腳本 |
| `Custom` | GET | `/Scripts/Custom` | 無 | JS 檔案 | 自訂腳本 |

## 關鍵邏輯

### Locale

1. 先從 `HttpContext.GetLocale()` 取得語系
2. 若有登入 Session，以 `clientInfo.Locale` 覆蓋
3. 透過 `MessageHelper(locale).GetScript()` 產生包含多語系訊息的 JS

### Main / Global

- 由 `GlobalFileProvider.GetGlobalFile("index"/"global", "js")` 讀取對應的全域 JS 檔案
- 檔案來源依方案設定（`ClientInfo`）

### Map / Custom

- `Map`：由 `GlobalProvider.GetMapScript()` 產生地圖初始化腳本
- `Custom`：由 `GlobalProvider.GetCustomScript()` 產生自訂腳本

## 備註

- 所有回傳均使用 `UTF8Encoding` 編碼，Content-Type 為 `application/javascript;charset=utf-8`
- 除 `Locale` 外，其餘 Action 皆需登入（檢查 Session），未登入回傳空內容
- `Locale` 有 fallback 機制，未登入時仍可產生預設語系腳本
