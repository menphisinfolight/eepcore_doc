# LogonController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/LogonController.cs` |
| 行數 | 43 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

處理使用者登入請求與驗證碼產生。為系統的主要登入入口。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| POST | `/Logon` | 提交登入表單 |
| GET/POST | `/Logon/RenderCaptcha` | 取得驗證碼圖片 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 處理登入請求 |
| `RenderCaptcha()` | GET | 無 | `IActionResult` | 產生並回傳驗證碼圖片 |

## 關鍵邏輯

### Index（登入）

- 透過 `AccountProvider.ProcessRequest()` 處理登入邏輯。
- 回傳值為字串時以 `Ok()` 回傳，否則以 `Json()` 回傳。

### RenderCaptcha（驗證碼）

1. 使用 `VerificationCodeHelper` 產生隨機驗證碼文字。
2. 將驗證碼存入 `Session`（透過 `SetCaptcha` 擴充方法）。
3. 產生驗證碼圖片的 byte 陣列，以 `application/octet-stream` 格式回傳 `captcha.png`。

## 備註

- 與 `AccountController` 共用 `AccountProvider.ProcessRequest()`，差異在於 `LogonController` 專門處理登入，不處理帳號管理頁面。
- `VerificationCodeHelper` 位於 `EEPBase.Core.Utility`。
