# SSOController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/SSOController.cs` |
| 行數 | 27 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

SSO（Single Sign-On）金鑰取得控制器，供外部系統透過 SSO 機制取得登入金鑰。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| POST | `/SSO` | 取得 SSO Key |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 取得 SSO 金鑰 |

## 關鍵邏輯

- 透過 `AccountProvider.GetSSOKey()` 處理 SSO 金鑰的產生與回傳。
- 回傳值為字串時以 `Ok()` 回傳，否則以 `Json()` 回傳。

## 備註

- 此控制器為通用 SSO 入口，與 `AzureSSOController`、`LdapSSOController` 為不同的 SSO 實作。
- `FlowAPIController` 中的 `GetSSOKey` 端點使用相同的 `AccountProvider.GetSSOKey()` 方法，功能等價。
