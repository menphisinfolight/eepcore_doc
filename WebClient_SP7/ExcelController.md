# ExcelController

> **檔案路徑**：`EEPWebClient.Core/Controllers/ExcelController.cs`（37 行）

## 用途

處理 Excel 檔案匯入，接收上傳的 `.xlsx` 檔案並透過 `FileProvider.ImportXlsx()` 解析內容。

## URL 路由

預設 MVC 路由：`/Excel/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | POST | `/Excel` | `IFormCollection`（含上傳的 Excel 檔案） | JSON / string | 匯入 Excel 並回傳解析結果 |

## 關鍵邏輯

1. 直接建立 `FileProvider` 並呼叫 `ImportXlsx(param)` 處理匯入
2. 未先檢查 Session 登入狀態（與 `ExcelfileController` 不同）
3. 回傳型別判斷：若結果為 string 則用 `Ok()`，否則用 `Json()` 序列化
4. 例外時回傳 `{error: message}` JSON

## 備註

- 與 `ExcelfileController` 功能類似，差異在於本控制器**未設定 `ClientInfo`**，`FileProvider` 會自行取得使用者資訊
- 可能為早期版本的 Excel 匯入端點，`ExcelfileController` 為改良版
