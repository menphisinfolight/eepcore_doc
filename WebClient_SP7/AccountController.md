# AccountController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/AccountController.cs` |
| 行數 | 73 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

帳號管理控制器，負責使用者註冊、帳號啟用、密碼重設與密碼變更等帳戶相關操作。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| GET | `/Account` | 帳號頁面（依 `type` 參數顯示不同功能） |
| GET | `/Account?p={token}` | 帳號啟用連結 |
| POST | `/Account` | 提交帳號操作表單 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index()` | GET | `[HttpGet, AddMessage, AddPublicKey, AddTheme]` | `IActionResult` | 顯示帳號頁面或執行帳號啟用 |
| `Index(IFormCollection)` | POST | 無 | `IActionResult` | 處理帳號操作請求 |

## 關鍵邏輯

### GET Index

1. **帳號啟用流程**：若 QueryString 帶有 `p` 參數，透過 `AccountProvider.ActivateUser()` 啟用使用者，回傳內嵌 `<script>` 的 HTML。
2. **頁面顯示流程**：依 `type` 查詢參數決定頁面標題：
   - `registerU` → 註冊使用者
   - `resetP` → 重設密碼
   - `changeP` → 變更密碼
3. 透過 `ConfigHelper.GetConfig()["registerUser"]` 讀取是否開放自行註冊的設定。

### POST Index

- 將表單參數交由 `AccountProvider.ProcessRequest()` 統一處理。
- 回傳值為字串時以 `Ok()` 回傳，否則以 `Json()` 回傳。

## 備註

- 依賴 `AccountProvider`（位於 `EEPGlobal.Core.Provider`）處理所有帳號邏輯。
- GET 方法掛載了 `AddMessage`、`AddPublicKey`、`AddTheme` 三個自訂 Filter，分別注入多語系訊息、公鑰與主題設定。
- `AddPublicKey` Filter 僅出現在此控制器，推測用於前端加密密碼傳輸。
