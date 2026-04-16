# RouteTransformer

| 項目 | 值 |
|------|-----|
| 檔案路徑 | `EEPWebClient.Core/Utility/RouteTransformer.cs` |
| 行數 | 135 行 |
| 命名空間 | `EEPWebClient.Core` |
| 繼承 | 所有類別繼承 `DynamicRouteValueTransformer` |

## 用途

此檔案定義了 EEP WebClient 所有的**動態路由轉換器**，將 URL 模式對應至特定的 Controller/Action。ASP.NET Core 的 `MapDynamicControllerRoute<T>` 在路由匹配時呼叫 `TransformAsync()`，回傳修改後的 `RouteValueDictionary`，讓框架分派至正確的端點。

## 路由轉換器一覽

| 類別 | URL 模式 | 目標 Controller | 目標 Action | 說明 |
|------|----------|-----------------|-------------|------|
| `RWDRouteTransformer` | `bootstrap/{page}` | Main | RenderPage | RWD 頁面渲染 |
| `DesignRouteTransformer` | `design/{page}` | Design | RenderPage | 設計模式頁面 |
| `DownloadFileRouteTransformer` | `userdesign/{fileType}/{fileName}` | UserDesign | DownloadFile | 使用者設計檔案下載 |
| `FlowRouteTransformer` | `flow/{page}` | Flow | RenderPage | 流程頁面渲染 |
| `CaptchaTransformer` | `captcha.png` | Logon | RenderCaptcha | 驗證碼圖片產生 |
| `MethodRouteTransformer` | `method/{name}` | Main | CallMethod | 伺服器方法呼叫 |
| `ProcRouteTransformer` | `proc/{name}` | Main | CallProc | 流程處理器方法呼叫 |
| `UserRouteTransformer` | `{route}` | Main | RoutePage | 自訂路由（萬用） |

## 實作細節

### 保留路由值的轉換器

以下轉換器**修改現有的 `RouteValueDictionary`**（`v = (RouteValueDictionary)obj`），保留原始路由參數（如 `page`、`fileType`、`fileName`）：

- `RWDRouteTransformer` — 保留 `{page}` 參數
- `DesignRouteTransformer` — 保留 `{page}` 參數
- `DownloadFileRouteTransformer` — 保留 `{fileType}` 與 `{fileName}` 參數
- `FlowRouteTransformer` — 保留 `{page}` 參數

### 建立新路由值的轉換器

以下轉換器**建立新的 `RouteValueDictionary`**（`v = new RouteValueDictionary()`），不保留 URL 模式中的路由值：

- `CaptchaTransformer` — 固定路由，無需參數
- `MethodRouteTransformer` — `{name}` 參數需從 URL 路徑手動取得
- `ProcRouteTransformer` — `{name}` 參數需從 URL 路徑手動取得
- `UserRouteTransformer` — `{route}` 參數需從 URL 路徑手動取得

> **注意**：`MethodRouteTransformer`、`ProcRouteTransformer`、`UserRouteTransformer` 建立新的 `RouteValueDictionary` 但未將原始路由參數（`name` / `route`）複製過去。ASP.NET Core 會從 URL 路徑重新解析這些參數，因此 Controller Action 仍能正確接收。

## 關鍵邏輯

1. **非同步包裝**：所有 `TransformAsync` 使用 `Task.Factory.StartNew()` 包裝同步邏輯，回傳 `ValueTask<RouteValueDictionary>`
2. **路由值傳遞方式**：兩種策略
   - 修改原始 `RouteValueDictionary`：簡潔，自動保留所有路由參數
   - 建立新 `RouteValueDictionary`：乾淨，僅設定 controller/action
3. **萬用路由**：`UserRouteTransformer` 匹配 `{route}`，作為最後的 fallback 路由，用於設定檔中定義的自訂路由頁面

## 備註

- 所有轉換器在 `Startup.ConfigureServices` 中以 `Singleton` 生命週期註冊
- 路由匹配順序由 `Startup.Configure` 中 `UseEndpoints` 的註冊順序決定，`UserRouteTransformer`（`{route}`）必須在最後，否則會攔截其他路由
- `CaptchaTransformer` 是唯一匹配固定路徑（`captcha.png`）的轉換器，其餘皆使用路由參數
- 此檔案雖放在 `Utility/` 目錄下，但在命名空間上屬於 `EEPWebClient.Core`（非 `Utility` 子命名空間）
