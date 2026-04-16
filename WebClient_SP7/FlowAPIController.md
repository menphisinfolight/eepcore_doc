# FlowAPIController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/FlowAPIController.cs` |
| 行數 | 134 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `ControllerBase` |
| 路由前綴 | `api/flow` |

## 用途

流程 Web API 控制器，提供外部系統透過 SSO Key 呼叫流程方法與查詢流程資料的 RESTful API 介面。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| POST | `/api/flow/GetSSOKey` | 取得 SSO 金鑰 |
| POST | `/api/flow/CallFlowMethod` | 呼叫流程方法 |
| POST | `/api/flow/QueryFlow` | 查詢流程資料 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `GetSSOKey()` | POST | `[Route("GetSSOKey"), HttpPost]` | `IActionResult` | 取得 SSO 金鑰 |
| `CallFlowMethod()` | POST | `[Route("CallFlowMethod"), HttpPost]` | `IActionResult` | 呼叫流程方法 |
| `QueryFlow()` | POST | `[Route("QueryFlow"), HttpPost]` | `object` | 查詢流程資料 |

## 關鍵邏輯

### GetSSOKey

- 呼叫 `AccountProvider.GetSSOKey()` 產生 SSO 金鑰。
- 回傳格式為 `{ key: "..." }` 的 JSON。
- 與 `SSOController` 功能等價，但包裝為 API 格式。

### CallFlowMethod

1. 從表單參數中取得 `key` 並呼叫 `Logon()` 進行 SSO 驗證。
2. 取得 Session 中的 `ClientInfo`，若為 null 則拋出 `"SSO Key is invalid."`。
3. 透過 `FlowHelper.CallFlowMethod()` 執行流程方法。
4. 流程目錄路徑為 `design/netflow/{solution}`。

### QueryFlow

1. 同樣先執行 SSO 驗證。
2. 透過 `FlowHelper.QueryFlow()` 查詢流程資料。

### Logon 私有方法

```csharp
private void Logon(IFormCollection param)
{
    var key = param["key"];
    if (string.IsNullOrEmpty(key))
        throw new Exception("SSO Key is null.");
    new AccountProvider(HttpContext).LogonWithKey(key);
}
```

所有需要驗證的 API 端點皆先呼叫此方法，以 SSO Key 完成登入。

### GetFlowDir 私有方法

- 回傳 `design/netflow/{solution}`，為流程定義檔的目錄路徑。

### Error 私有方法

- 統一錯誤回傳格式：`{ error: "..." }`。

## 備註

- 此控制器標記為 `[ApiController]`，繼承 `ControllerBase`（非 `Controller`），不支援 View 渲染。
- 所有端點皆為 POST，使用 SSO Key 進行身分驗證（非 Session Cookie 驗證）。
- `FlowHelper` 位於 `EEPServerTools.Core.Utility`，為流程操作的核心工具類別。
- 流程目錄固定使用 `design/netflow/` 前綴，表示此 API 專用於 EEP.NET Flow。
