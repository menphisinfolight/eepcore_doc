# AzureSSOController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/AzureSSOController.cs` |
| 行數 | 175 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

Azure AD SSO 登入控制器，實作 OAuth 2.0 Authorization Code 流程，與 Microsoft Entra ID（Azure AD）整合完成單一登入。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| GET | `/AzureSSO` | 發起 Azure AD 登入或處理回調 |
| GET/POST | `/AzureSSO/AzureCheck` | 驗證 Azure 使用者（第二種登入模式） |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index()` | GET | 無 | `Task<IActionResult>` | Azure AD OAuth 流程（重導向 + 回調處理） |
| `AzureCheck(...)` | GET/POST | 無 | `IActionResult` | 以 userid + code 驗證 Azure 使用者 |

## 關鍵邏輯

### Index — OAuth 2.0 Authorization Code Flow

1. **讀取設定**：從 `config["AzureSSOs"]` 陣列讀取 `TenantID`、`ClientID`、`ClientSecret`、`Database`、`Solution`，存入靜態類別 `AzureSSOVars`。
2. **無 code 參數（第一次請求）**：
   - 組合 `callback_url` 為 `https://{host}/azuresso`。
   - 驗證五項設定皆不為空，否則顯示錯誤。
   - 重導向至 Microsoft 授權端點 `login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize`，scope 為 `user.read`。
3. **有 code 參數（回調）**：
   - 以 `authorization_code` grant type 向 Microsoft token 端點換取 `access_token`。
   - 使用 access_token 呼叫 `graph.microsoft.com/v1.0/me` 取得使用者 email。
   - 處理外部使用者（`#EXT#`）的 email 格式轉換。
   - 呼叫 `AccountProvider.LogonAzure()` 完成 EEP 系統登入。
   - 成功後重導向至 `../main`。

### AzureCheck — 替代驗證模式

- 接收 `userid`、`code`、`database`、`solution` 參數。
- 驗證 database 與 solution 與設定一致後，呼叫 `AccountProvider.LogonAzure2()` 登入。
- 成功回傳重導向路徑 `main?database={database}&solution={solution}`。

### AzureSSOVars 靜態類別

儲存 Azure SSO 設定值的靜態容器（定義於同檔案第 166-174 行）：

| 欄位 | 說明 |
|------|------|
| `tenant_id` | Azure AD 租戶 ID |
| `client_id` | 應用程式用戶端 ID |
| `client_secret` | 應用程式密鑰 |
| `callback_url` | OAuth 回調 URL |
| `solution` | EEP 方案名稱 |
| `database` | EEP 資料庫名稱 |

## 備註

- 使用靜態變數 `AzureSSOVars` 儲存設定，在多執行緒/多使用者同時登入時有潛在競爭風險。
- 外部使用者 email 格式處理：`user_domain.com#EXT#@tenant.onmicrosoft.com` → `user@domain.com`。
- 錯誤情境（設定缺失、token 取得失敗、使用者未找到、帳號停用）皆透過 `ViewBag.ErrorMessage` 傳至 View 顯示。
