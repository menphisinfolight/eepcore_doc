# DesignController

| 項目 | 值 |
|------|-----|
| 檔案路徑 | `EEPWebClient.Core/Controllers/DesignController.cs` |
| 行數 | 90 行 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

DesignController 負責 EEP 的**設計模式（Design Mode）**，提供表單設計器、資料模組設計等開發工具的入口。僅限開發者（`clientInfo.Dev == true`）存取。同時處理初始化管理員與資料庫連線設定。

## URL 路由

| URL 模式 | HTTP 方法 | 對應 Action | 路由來源 |
|-----------|-----------|-------------|----------|
| `/Design` 或 `/Design/Index` | GET | `Index()` | 預設路由 |
| `/Design` 或 `/Design/Index` | POST | `Index(IFormCollection)` | 預設路由 |
| `/design/{page}` | GET | `RenderPage(page)` | `DesignRouteTransformer` |

## Actions / 方法表

### Index (GET) — 設計模式入口

```csharp
[HttpGet, AddMessage, AddPublicKey, AddClientInfo, AddTheme]
public IActionResult Index()
```

**分流邏輯：**

| 條件 | 回傳 View |
|------|-----------|
| `clientInfo != null && clientInfo.Dev` | `View()`（設計器主頁面） |
| `DatabaseProvider.InitKey()` 回傳 `true` | `View("Admin")`（管理員初始化頁面） |
| 其他 | `View("Logon")`（開發者登入頁面） |

**Filter Attributes：**
- `AddMessage`：注入系統訊息
- `AddPublicKey`：注入公開金鑰（用於加密傳輸）
- `AddClientInfo`：注入客戶端資訊
- `AddTheme`：注入主題設定

### Index (POST) — 設計操作分派

```csharp
[HttpPost]
public IActionResult Index(IFormCollection param)
```

**兩種分派路徑：**

1. **資料庫初始化**（`param["type"] == "database" && param["mode"] == "saveAdmin"`）
   - 呼叫 `DatabaseProvider.CheckAndInit(key, datas)`
   - 成功回傳 `Ok()`，失敗回傳錯誤訊息

2. **一般設計操作**
   - 驗證 `clientInfo.Dev`，否則拋出 `EEPException("Timeout")`
   - 透過 `DesignProvider.CreateProvider(HttpContext, param["type"])` 根據 `type` 建立對應 Provider
   - 呼叫 `provider.ProcessRequest(param)` 處理請求
   - 回傳邏輯同 MainController（string → Ok, 其他 → Json）

### RenderPage — 設計頁面渲染

```csharp
[AddMessage]
public IActionResult RenderPage(string page)
```

- 由 `DesignRouteTransformer` 將 `/design/{page}` 路由至此
- 驗證 `clientInfo.Dev`，否則拋出 `EEPException("DevTimeout")`
- 直接使用 MVC `View(page)` 渲染（設計工具頁面為預先定義的 Razor View，非動態 RWD 頁面）

## 關鍵邏輯

1. **雙重身份驗證**：設計模式要求 `clientInfo.Dev == true`，一般使用者無法進入
2. **Provider 工廠模式**：`DesignProvider.CreateProvider()` 根據 `param["type"]` 建立不同 Provider（如表單設計、資料模組設計、流程設計等）
3. **資料庫初始化流程**：系統首次部署時，透過 `DatabaseProvider` 完成管理員帳號與資料庫連線設定
4. **與 MainController 的差異**：RenderPage 使用 Razor View 而非 `RWDPage.Render()`，因為設計工具介面是靜態定義的

## 備註

- `clientInfo` 從 `HttpContext.Session.GetClientInfo()` 取得（非 AccountProvider），表示設計模式使用 Session-based 驗證
- 初始化路徑（Admin View）僅在 `DatabaseProvider.InitKey()` 成功時顯示，此為系統首次安裝的引導流程
- `DesignProvider.CreateProvider` 的 `type` 參數決定了後端處理的設計模組類別
