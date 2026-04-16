# FlowController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/FlowController.cs` |
| 行數 | 117 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

流程引擎主控制器，負責流程方法呼叫與流程相關頁面渲染。自動判斷使用 EEP.NET Flow 或傳統 Flow Provider。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| POST | `/Flow` | 呼叫流程方法 |
| GET | `/Flow/RenderPage/{page}` | 渲染流程相關頁面 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 處理流程操作請求 |
| `RenderPage(string page)` | GET | `[AddMessage, AddClientInfo, AddTheme]` | `IActionResult` | 渲染指定流程頁面 |

## 關鍵邏輯

### Provider 屬性

```csharp
private BaseProvider Provider =>
    EEPNetHelper.IsEEPNETFlow
        ? new EEPNetFlowProvider(HttpContext)
        : new FlowProvider(HttpContext);
```

依 `EEPNetHelper.IsEEPNETFlow` 旗標自動選擇 Provider，實現 EEP.NET Flow 與傳統 Flow 的統一入口。

### Index（流程操作）

1. 從 Session 取得 `ClientInfo`，若為 null 則拋出 `EEPException("MainTimeout")`。
2. 依 Provider 屬性取得對應的 FlowProvider 或 EEPNetFlowProvider。
3. 呼叫 `provider.ProcessRequest(param)` 處理請求。

### RenderPage（頁面渲染）

依 `page` 參數分三種情境：

#### 1. `page == "mail"`（流程郵件連結）
- 呼叫 `FlowProvider.LogonWithKey()` 以金鑰登入。
- 若設定 `config["ssoAD"] == "true"`，執行 LDAP 二次驗證：
  - 從 `User.Identity.Name` 解析 domain 與 userid。
  - 比對 Session 中的使用者是否與 Windows 驗證使用者一致。
  - 不一致則清除 Session 並拋出 `"AD Validate failed."`。
  - 一致則重新執行 `AccountProvider.LogonLdap()` 登入。
- 將 `mailParam` 傳入 ViewBag 後渲染 mail View。
- 登入失敗則重導向至首頁 `../`。

#### 2. `page == "netpreview"`
- 設定 `ViewBag.isEEP = "true"`，渲染 `preview` View。

#### 3. 其他頁面
- 設定 `ViewBag.isEEP` 依 `EEPNetHelper.IsEEPNETFlow` 判斷。
- 渲染對應名稱的 View。

## 備註

- 依賴 `FlowProvider`（`EEPGlobal.Core.Provider`）與 `EEPNetFlowProvider`（同命名空間）。
- `mail` 頁面的 SSO AD 驗證邏輯與 `LdapSSOController` 類似，但多了使用者身分比對步驟。
- `netpreview` 頁面強制以 EEP 模式（`isEEP = "true"`）渲染 `preview` View。
