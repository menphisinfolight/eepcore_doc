# Startup

| 項目 | 值 |
|------|-----|
| 檔案路徑 | `EEPWebClient.Core/Startup.cs` |
| 行數 | 173 行 |
| 命名空間 | `EEPWebClient` |
| 類別 | `Startup` |

## 用途

Startup 是 ASP.NET Core 應用程式的啟動設定類別，負責**服務註冊（DI）**與**中介軟體管線（Middleware Pipeline）**的組態。定義了所有路由規則、Session 設定、多語系支援及動態路由轉換器的註冊。

## ConfigureServices — 服務註冊

### DI 註冊項目

| 註冊項目 | 生命週期 | 說明 |
|----------|----------|------|
| `RWDRouteTransformer` | Singleton | `/bootstrap/{page}` 路由轉換 |
| `DesignRouteTransformer` | Singleton | `/design/{page}` 路由轉換 |
| `DownloadFileRouteTransformer` | Singleton | `/userdesign/{fileType}/{fileName}` 路由轉換 |
| `FlowRouteTransformer` | Singleton | `/flow/{page}` 路由轉換 |
| `CaptchaTransformer` | Singleton | `/captcha.png` 路由轉換 |
| `MethodRouteTransformer` | Singleton | `/method/{name}` 路由轉換 |
| `ProcRouteTransformer` | Singleton | `/proc/{name}` 路由轉換 |
| `UserRouteTransformer` | Singleton | `/{route}` 萬用路由轉換 |
| `SessionAttribute` | MVC Filter | 全域 Session 過濾器 |

### Session 設定

- Cookie 名稱：`.AspNetCore.Session.Design4438`
- 可選 SQL Server 分散式快取（由 `Configuration["SqlServerCache"]` 控制）
  - 連線字串：`Configuration["SqlSeverConnectionString"]`
  - 資料表：`dbo.AspNetCoreSession`

### 表單上傳限制

- `ValueCountLimit`：`int.MaxValue`（無限制）
- `ValueLengthLimit`：`int.MaxValue`（支援超過 5MB 的 blob 資料）

### 多語系設定

- 支援語系：`en-US`、`zh-TW`、`zh-CN`
- 預設語系：`zh-TW`
- 語系判斷方式：`AcceptLanguageHeaderRequestCultureProvider`（根據瀏覽器 Accept-Language header）

## Configure — 中介軟體管線

管線執行順序（由上而下）：

```
1. UseExceptionHandler      → 全域例外處理（回傳 text/html 錯誤訊息）
2. UseRequestLocalization    → 多語系本地化
3. CORS middleware (inline)  → 加入 Access-Control-Allow-Origin: *
4. UseHttpsRedirection       → HTTPS 重導
5. UseStaticFiles            → 靜態檔案服務
6. UseRouting                → 路由中介軟體
7. UseSession                → Session 中介軟體
8. UseAuthorization          → 授權中介軟體
9. UseEndpoints              → 端點對應（路由規則）
10. UseStaticFiles (第二次)   → 特殊 MIME 類型靜態檔案
```

### 路由規則（UseEndpoints）

| 順序 | URL 模式 | 路由轉換器 | 目標 Controller/Action |
|------|----------|-----------|----------------------|
| 1 | `bootstrap/{page}` | `RWDRouteTransformer` | `Main/RenderPage` |
| 2 | `design/{page}` | `DesignRouteTransformer` | `Design/RenderPage` |
| 3 | `userdesign/{fileType}/backup/{fileName}` | `DownloadFileRouteTransformer` | `UserDesign/DownloadFile` |
| 4 | `userdesign/{fileType}/{fileName}` | `DownloadFileRouteTransformer` | `UserDesign/DownloadFile` |
| 5 | `flow/{page}` | `FlowRouteTransformer` | `Flow/RenderPage` |
| 6 | `method/{name}` | `MethodRouteTransformer` | `Main/CallMethod` |
| 7 | `proc/{name}` | `ProcRouteTransformer` | `Main/CallProc` |
| 8 | `captcha.png` | `CaptchaTransformer` | `Logon/RenderCaptcha` |
| 9 | `{controller=Home}/{action=Index}/{id?}` | 無（預設路由） | 依 URL 決定 |
| 10 | `{route}` | `UserRouteTransformer` | `Main/RoutePage` |

**路由優先序**：動態路由按註冊順序比對，`{route}` 為萬用路由放在最後，作為 fallback。

### 特殊 MIME 設定

- `.properties` → `application/octet-stream`

## 關鍵邏輯

1. **動態路由轉換器架構**：使用 ASP.NET Core 的 `MapDynamicControllerRoute<T>` 搭配自訂 `DynamicRouteValueTransformer`，在執行時期動態決定目標 Controller/Action
2. **雙層靜態檔案服務**：第一次 `UseStaticFiles()` 處理一般靜態檔案，第二次加上自訂 MIME 映射處理 `.properties` 等特殊格式
3. **CORS 簡易實作**：直接以 inline middleware 加入 `Access-Control-Allow-Origin: *`，而非使用 ASP.NET Core 內建 CORS 框架（已有完整 CORS policy 的註解程式碼保留供參考）
4. **Session 作為分散式快取**：支援 SQL Server 分散式 Session，適用於多節點部署場景

## 備註

- 註解中有完整的 CORS policy 設定（`CorsPolicyName0519`），目前未啟用，改以簡易 inline middleware 處理
- `UseStaticFiles` 出現兩次：第一次在 `UseRouting` 之前，第二次在 `UseEndpoints` 之後，第二次專門處理特殊副檔名
- `SessionAttribute` 作為全域 MVC Filter 註冊，所有 Controller Action 都會經過 Session 檢查
- SQL Server Session 快取的連線字串金鑰拼寫為 `SqlSeverConnectionString`（少了一個 'r'），這是原始碼中的拼字
