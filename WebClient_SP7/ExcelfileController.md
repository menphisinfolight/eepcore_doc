# ExcelfileController

> **檔案路徑**：`EEPWebClient.Core/Controllers/ExcelfileController.cs`（41 行）

## 用途

處理 Excel 檔案匯入，與 `ExcelController` 類似但會明確從 Session 取得 `ClientInfo` 並設定給 `FileProvider`。

## URL 路由

預設 MVC 路由：`/Excelfile/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | POST | `/Excelfile` | `IFormCollection`（含上傳的 Excel 檔案） | JSON / string | 匯入 Excel 並回傳解析結果 |

## 關鍵邏輯

1. 從 `HttpContext.Session.GetClientInfo()` 取得使用者資訊
2. 建立 `FileProvider` 並明確設定 `provider.ClientInfo = clientInfo`
3. 呼叫 `provider.ImportXlsx(param)` 解析 Excel 內容
4. 回傳型別判斷：string 用 `Ok()`，否則用 `Json()`
5. 例外時回傳 `{error: message}` JSON

## 備註

- 與 `ExcelController` 的差異：本控制器**明確設定 `ClientInfo`**，確保 `FileProvider` 能正確取得方案路徑等資訊
- 原始碼中保留了與 `ExcelController` 相同寫法的註解（`//var result = new FileProvider(HttpContext).ImportXlsx(param);`），顯示為改良版本
