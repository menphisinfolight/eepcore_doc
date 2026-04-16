# Security

> `EEPRWDTools.Core/Controls/Security.cs` — 18 行
> 繼承：`RWDControl` → `Component`

## 用途

**頁面存取權限標記元件**（Security）。

Security 是一個**標記型元件**，不產生任何 HTML 輸出（Render 方法為空），也沒有前端 JavaScript。它的唯一作用是**告訴 RWDPage 這個頁面需要進行選單權限檢查**。

## 運作機制

在 `RWDPage.LoadControls()` 中（`RWDPage.cs` line 180）：

```csharp
if (!ClientInfo.Dev && Controls.OfType<Security>().Count() > 0)
{
    // 檢查是否有合法的加密參數（直連 URL 的驗證）
    if (Query.ContainsKey("p") && Query.ContainsKey("pm"))
    {
        var md5 = MD5(Query["p"]);
        if (md5 == Query["pm"])
        {
            return;  // 加密驗證通過，允許存取
        }
    }

    // 檢查選單權限
    if (menuInfo["id"] == null)
    {
        throw new EEPException("accessDenied");  // 無權限，拒絕存取
    }
}
```

### 判斷邏輯

```text
頁面載入
  → 檢查是否有 Security 元件
    → 沒有 → 任何人都能存取（公開頁面）
    → 有 → 進入權限檢查：
      → 開發模式（Dev）→ 跳過檢查
      → 有加密參數（p + pm）且 MD5 驗證通過 → 允許存取
      → 查詢 runtimeMenu 選單權限：
        → menuInfo["id"] 存在 → 使用者有此選單權限，允許存取
        → menuInfo["id"] 為 null → 拋出 "accessDenied" 例外，拒絕存取
```

### MenuInfo 查詢

```csharp
// RWDPage.cs - MenuInfo 屬性
var table = new DataModule(ClientInfo, DataModule.SYSTEM_TABLE)
    .GetDataset("runtimeMenu", new QueryOptions() { DatabaseType = DatabaseType.Split });

// 在 runtimeMenu 結果中尋找匹配的選單項目
// runtimeMenu 已經過 USERMENUS ∪ GROUPMENUS 的權限過濾
```

`runtimeMenu` 只會回傳使用者有權限的選單（透過 USERMENUS 和 GROUPMENUS 過濾），如果找不到對應的選單項目，`menuInfo["id"]` 就是 null，代表使用者無權存取此頁面。

## 加密參數存取（p + pm）

這是用於**不經過選單直接開啟頁面**的場景（如 Email 連結、外部系統跳轉）：

| 參數 | 說明 |
|------|------|
| `p` | 原始驗證字串 |
| `pm` | `p` 的 MD5 雜湊值（大小寫不敏感） |

URL 範例：`/bootstrap/OrderDetail?p=secret123&pm=F561AEB81F027C9B...`

只要 `MD5(p) == pm` 驗證通過，就允許存取，不需要選單權限。

## 設計介面

Security 元件**沒有任何屬性**可設定。從工具箱拖入頁面即可。

```json
{
  "type": "security",
  "id": "Security1"
}
```

## 有 Security vs 沒有 Security

| 情境 | 有 Security 元件 | 沒有 Security 元件 |
|------|:---------------:|:-----------------:|
| 登入使用者有選單權限 | ✅ 可存取 | ✅ 可存取 |
| 登入使用者無選單權限 | ❌ accessDenied | ✅ 可存取 |
| 未登入使用者 | ❌ 拒絕（Session 無 ClientInfo） | ✅ 可存取（若不需登入） |
| 開發模式（Dev） | ✅ 跳過檢查 | ✅ 可存取 |
| 加密參數驗證通過 | ✅ 可存取 | ✅ 可存取 |

## 備註

- Security 是 EEP Core 頁面權限的**唯一開關**。沒有放 Security 元件的頁面 = 公開頁面，任何人只要知道 URL 就能存取。
- Render 方法為空，不輸出任何 HTML，不佔頁面空間。
- 前端也沒有對應的 `$.createObj`，純粹是伺服端標記。
- 選單權限由 `runtimeMenu_onBeforeExecuteSQL` 過濾，背後查詢 USERMENUS 和 GROUPMENUS。
- 開發模式下 Security 檢查會被跳過（`ClientInfo.Dev = true`），方便開發測試。
