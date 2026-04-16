# LdapSSOController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/LdapSSOController.cs` |
| 行數 | 56 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

LDAP / Windows 驗證 SSO 登入控制器，利用 IIS Windows Authentication 取得的 AD 使用者身分自動完成 EEP 系統登入。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| GET | `/LdapSSO` | LDAP SSO 登入 |
| GET | `/LdapSSO/Test` | 測試目前 Windows 驗證使用者 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index()` | GET | 無 | `IActionResult` | LDAP SSO 登入 |
| `Test()` | GET | 無 | `IActionResult` | 回傳目前 Windows 驗證身分 |

## 關鍵邏輯

### Index

1. 從 QueryString 取得 `solution` 與 `database` 參數。
2. 從 `User.Identity.Name` 解析 `domain` 與 `userid`（格式為 `DOMAIN\userid`）。
3. 驗證四項參數（domain、userid、database、solution）皆不為空。
4. 呼叫 `AccountProvider.LogonLdap(domain, userid, database, solution)` 執行登入。
5. 登入成功（回傳空字串）→ 重導向至 `main`。
6. 登入失敗 → 回傳錯誤訊息。

### Test

- 單純回傳 `"Current User:" + User.Identity.Name`，用於偵錯 Windows 驗證是否正常。

## 備註

- 此控制器依賴 IIS 的 Windows Authentication 模組，`User.Identity.Name` 必須由伺服器端 Windows 驗證提供。
- 與 `FlowController.RenderPage("mail")` 中的 LDAP 驗證邏輯類似，但該處是用於流程郵件連結的 AD 二次驗證。
