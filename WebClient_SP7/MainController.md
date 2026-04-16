# MainController

| 項目 | 值 |
|------|-----|
| 檔案路徑 | `EEPWebClient.Core/Controllers/MainController.cs` |
| 行數 | 574 行 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

MainController 是 EEP WebClient 的**核心控制器**，負責處理所有運行時期（Runtime）的頁面渲染、資料操作、方法呼叫、檔案處理及路由分派。前端幾乎所有 AJAX 請求與頁面載入都經由此控制器。

## URL 路由

| URL 模式 | HTTP 方法 | 對應 Action | 路由來源 |
|-----------|-----------|-------------|----------|
| `/Main` 或 `/Main/Index` | GET | `Index()` | 預設路由 |
| `/Main` 或 `/Main/Index` | POST | `Index(IFormCollection)` | 預設路由 |
| `/bootstrap/{page}` | GET | `RenderPage(page)` | `RWDRouteTransformer` |
| `/method/{name}` | GET/POST (Form) | `CallMethodForm(name)` | `MethodRouteTransformer` |
| `/method/{name}` | POST (JSON) | `CallMethodJSON(name)` | `MethodRouteTransformer` |
| `/proc/{name}` | GET/POST | `CallProc(name)` | `ProcRouteTransformer` |
| `/Main/File` | POST | `File(IFormCollection)` | 預設路由 |
| `/Main/userDefFile` | POST | `userDefFile(IFormCollection)` | 預設路由 |
| `/{route}` | GET | `RoutePage(route)` | `UserRouteTransformer` |

## Actions / 方法表

### Index (GET)

```csharp
[HttpGet]
public IActionResult Index()
```

- 將 GET 請求重導至上一層路徑（`../`），並保留 QueryString。
- 用途：防止使用者直接 GET 存取 `/Main`。

### Index (POST) — 主要資料分派入口

```csharp
[HttpPost]
public IActionResult Index(IFormCollection param)
```

這是前端 AJAX 呼叫的**統一入口**，根據 `param["mode"]` 分派至 `DataProvider.ProcessRequest()`。

**免登入驗證的 mode（`nonLogonModes`）：**

| mode | 說明 |
|------|------|
| `getDataset` | 取得資料集 |
| `checkSession` | 檢查 Session 是否存活 |
| `updateDataset` | 更新資料集 |
| `callMethod` | 呼叫伺服器端方法 |
| `callProcessorMethod` | 呼叫流程處理器方法 |

**驗證邏輯：**
- 上述 mode 使用 `Referer` header 取得來源 URL
- `loadHtml` mode 使用 `param["page"]` 取得 URL
- 透過 `AccountProvider.GetClientInfo(url)` 驗證身份
- 若 `clientInfo == null` 則拋出 `EEPException("Timeout")`

**回傳邏輯：**
- 結果為 `string` → `Ok(result)`
- 結果為其他型別 → `Json(result)`

### RenderPage — 頁面渲染引擎

```csharp
[HttpGet, AddMessage, AddTheme]
public IActionResult RenderPage(string page)
```

由 `RWDRouteTransformer` 將 `/bootstrap/{page}` 路由至此。根據副檔名分派不同渲染邏輯：

| 副檔名 | 處理方式 |
|--------|----------|
| `.net` | 透過 `EEPNetHelper.GetSSOUrl()` 取得 SSO 網址，302 重導 |
| `.netflow` | 透過 `EEPNetFlowHelper.GetSSOUrl()` 取得 SSO 網址，302 重導 |
| `.js` | 透過 `RWDPage.RenderScript()` 產生 JavaScript，回傳 `application/javascript` |
| 無副檔名（一般頁面） | 透過 `RWDPage.Render()` 產生 HTML，回傳 `text/html` |

**系統管理頁面特殊處理：**
- 若 `page` 以 `sys` 或 `agentsetting` 開頭，使用 MVC View 渲染（`return View(page)`）
- 針對 `sysgroups`、`sysusers`、`sysagent`、`sysmenus`、`sysorg`，會檢查 `runtimeMenu` 資料表中是否有該頁面的權限，無權限回傳 404

**日誌記錄：**
- 成功時記錄 `LogType.Normal`
- 錯誤時記錄 `LogType.Error`（含 e.Message）

### CallMethodForm — 表單方式呼叫 Server 方法

```csharp
public IActionResult CallMethodForm(string name)
```

- 支援 GET（Query）與 POST（Form）
- 將參數轉為 `JObject` 後委派給 `CallMethod()`

### CallMethodJSON — JSON 方式呼叫 Server 方法

```csharp
public IActionResult CallMethodJSON(string name, [FromBody] JsonElement json)
```

- 接受 JSON body
- 解析後委派給 `CallMethod()`

### CallMethod（private）— 方法呼叫核心

```csharp
private IActionResult CallMethod(string name, JObject values)
```

- `name` 格式：`{module}.{method}`（如 `OrderModule.GetList`）
- 組裝 `mode=callMethod`、`module`、`method`、`parameters` 後交由 `DataProvider.ProcessRequest()` 處理
- 成功回傳 `{ "result": ... }`
- 失敗回傳 `{ "error": "..." }`

### CallProc — 呼叫流程處理器方法

```csharp
public IActionResult CallProc(string name)
```

- 支援 GET 與 POST
- 組裝 `mode=callProcessorMethod`、`id={name}`、`parameters` 後交由 `DataProvider.ProcessRequest()` 處理
- 回傳格式同 `CallMethod`

### File — 檔案操作

```csharp
[HttpPost]
public IActionResult File(IFormCollection param)
```

- 透過 `FileProvider.ProcessRequest()` 處理檔案上傳/下載等操作
- 使用 `Referer` header 驗證身份

### userDefFile — 使用者自訂檔案上傳

```csharp
[HttpPost]
public IActionResult userDefFile(IFormCollection param)
```

- 支援 `.docx`（Word）與 `.xlsx`（Excel）檔案
- 先呼叫 `ImportUserDefFile()` 儲存檔案至 `./design/doc/`
- 再組裝 `mode=addUserDefFile` 交由 `DataProvider.ProcessRequest()` 處理
- 不支援的副檔名回傳錯誤

### ImportUserDefFile — 檔案儲存輔助方法

```csharp
public string ImportUserDefFile(IFormCollection param)
```

- 透過 `FileProvider.GetFilePath()` 取得儲存路徑
- 將上傳檔案寫入磁碟，回傳檔案路徑

### RoutePage — 自訂路由頁面

```csharp
[HttpGet]
public IActionResult RoutePage(string route)
```

- 從 `ConfigHelper.GetConfig()` 讀取 `routes` 設定
- 比對 `route` 名稱，找到後以 `RWDPage.RenderIframe(url)` 渲染 iframe 頁面
- 若 URL 非 http(s) 開頭，自動加上 `../` 前綴
- 找不到對應路由回傳 404

### ConvertToBootstrap5 — Bootstrap 3 → 5 轉換

```csharp
public string ConvertToBootstrap5(string bootstrap3Html)
```

目前被註解未使用（`RenderPage` 中的 `ConvertToBootstrap5` 呼叫已註解），但完整實作了以下轉換：

| 轉換項目 | 子方法 |
|----------|--------|
| 網格系統（col-xs → col, offset） | `ConvertGridSystem` |
| 按鈕（btn-default → btn-secondary, glyphicon → bi） | `ConvertButtons` |
| 表單（form-group → mb-3 等） | `ConvertForms` |
| 導航列（navbar-default → navbar-light 等） | `ConvertNavbar` |
| 警告框（dismissable → dismissible） | `ConvertAlerts` |
| 面板 → 卡片（panel → card） | `ConvertPanelsToCards` |
| 通用工具類別（visible/hidden, 間距） | `ConvertUtilityClasses` |
| 分頁元件 | `AdjustPagination` |
| 間距與對齊 | `AdjustSpacingAndAlignment` |

## 關鍵邏輯

1. **Provider 分派模式**：POST Index 是統一入口，所有前端操作透過 `param["mode"]` 決定行為，實際邏輯在 `DataProvider.ProcessRequest()` 中
2. **身份驗證**：透過 `AccountProvider.GetClientInfo()` 取得 `ClientInfo`，來源可為 URL（Referer / page 參數）或 JObject
3. **RenderPage 多副檔名分流**：`.net` / `.netflow` 走 SSO 重導，`.js` 回傳腳本，一般頁面走 `RWDPage.Render()` 產生 HTML
4. **系統頁面權限檢查**：sys 開頭的頁面額外檢查 `runtimeMenu` 資料表權限
5. **錯誤處理**：API 方法（CallMethod / CallProc）捕獲例外回傳 JSON error；RenderPage 的 Session 逾時回傳 Timeout View

## 備註

- `ConvertToBootstrap5` 系列方法已完成但未啟用，為 Bootstrap 3 → 5 遷移的準備工作
- `CallMethodForm` / `CallMethodJSON` 兩個 public action 分別處理 Form 與 JSON 格式，最終都匯入 private `CallMethod`
- `MethodRouteTransformer` 路由到 action 名稱 `CallMethod`，但實際 public action 名稱為 `CallMethodForm` / `CallMethodJSON`，由 ASP.NET Core 根據請求 Content-Type 與參數綁定自動匹配
- `nonLogonModes` 並非「免登入」，而是使用 Referer 取得 URL 進行身份驗證（與 `loadHtml` 使用 `param["page"]` 不同）
