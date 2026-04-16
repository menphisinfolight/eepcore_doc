# HomeController

| 項目 | 值 |
|------|-----|
| 檔案路徑 | `EEPWebClient.Core/Controllers/HomeController.cs` |
| 行數 | 60 行 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

HomeController 是 EEP WebClient 的**首頁控制器**，負責使用者登入判斷與主頁面載入。當使用者存取網站根路徑時，由此控制器決定顯示登入頁面或主畫面。

## URL 路由

| URL 模式 | HTTP 方法 | 對應 Action | 路由來源 |
|-----------|-----------|-------------|----------|
| `/` 或 `/Home` 或 `/Home/Index` | GET | `Index()` | 預設路由（`{controller=Home}/{action=Index}`） |

## Actions / 方法表

### Index (GET) — 網站入口

```csharp
[HttpGet, AddMessage, AddPublicKey, AddClientInfo, AddTheme,
 AddMainBackground, AddLogonInfo, AddStartPage, AddRWDRef]
public IActionResult Index()
```

**Filter Attributes（共 8 個）：**

| Attribute | 功能 |
|-----------|------|
| `AddMessage` | 注入系統訊息 |
| `AddPublicKey` | 注入 RSA 公開金鑰（前端加密密碼用） |
| `AddClientInfo` | 注入客戶端資訊至 ViewBag |
| `AddTheme` | 注入佈景主題設定 |
| `AddMainBackground` | 注入主畫面背景圖設定 |
| `AddLogonInfo` | 注入登入頁資訊（如公司名稱、Logo 等） |
| `AddStartPage` | 注入起始頁面設定 |
| `AddRWDRef` | 注入 RWD 相關資源參考（CSS/JS） |

**分流邏輯：**

```
AccountProvider.LogonWithKey()
  ├─ 拋出例外 → View("Logon")（登入頁）
  ├─ 回傳非空字串 → RedirectResult(redirectUrl)（SSO 或外部重導）
  ├─ 回傳 null → View("Logon")（登入頁）
  └─ 回傳空字串 → 檢查 Session
       ├─ clientInfo == null → View("Logon")
       ├─ clientInfo.LogonTime == null → View("Logon")
       └─ 已登入 → View()（主畫面）
```

**主畫面 ViewBag 注入：**
- `ViewBag.IsBothFlow`：是否同時支援 .NET 與 .NET Flow 雙流程
- `ViewBag.IsFlowWarning`：是否顯示流程警告

## 關鍵邏輯

1. **LogonWithKey() 自動登入**：嘗試以金鑰自動登入（SSO 場景），失敗則顯示登入頁
2. **多層 Logon 判斷**：例外 → null → 空字串 → Session 狀態 → LogonTime，任一層不通過即回到登入頁
3. **8 個 Filter Attributes**：為登入頁與主頁面注入所有必要的前端資源與設定資料
4. **僅 GET**：此控制器只有 GET 動作，登入的 POST 請求由 `LogonController` 處理

## 備註

- 此控制器為預設路由的預設控制器（`{controller=Home}`），是使用者進入系統的第一個端點
- `AccountProvider.LogonWithKey()` 支援 QueryString 中的金鑰自動登入，常用於 SSO 或郵件連結場景
- 登入表單的 POST 處理不在此控制器中，而是由獨立的 LogonController 負責
- `EEPNetHelper.IsBothFlow` / `IsFlowWarning` 為靜態屬性，用於判斷系統是否啟用雙流程引擎
