# NetFlowController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/NetFlowController.cs` |
| 行數 | 36 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

EEP.NET Flow 專用控制器，處理 EEP.NET 流程引擎的操作請求。與 `FlowController` 功能類似，但固定使用 `EEPNetFlowProvider`。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| POST | `/NetFlow` | 處理 EEP.NET 流程操作 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 處理 EEP.NET 流程請求 |

## 關鍵邏輯

### Index

1. 從 Session 取得 `ClientInfo`，若為 null 則拋出 `EEPException("MainTimeout")`。
2. 建立 `EEPNetFlowProvider` 並設定 `ClientInfo`。
3. 呼叫 `provider.ProcessRequest(param)` 處理請求。
4. 回傳值為字串時以 `Ok()` 回傳，否則以 `Json()` 回傳。

## 備註

- 與 `FlowController` 的差異：`FlowController` 會依 `EEPNetHelper.IsEEPNETFlow` 自動選擇 Provider，而 `NetFlowController` 固定使用 `EEPNetFlowProvider`。
- 此控制器不含頁面渲染功能，頁面渲染統一由 `FlowController.RenderPage()` 處理。
- `EEPNetFlowProvider` 位於 `EEPGlobal.Core.Provider`。
