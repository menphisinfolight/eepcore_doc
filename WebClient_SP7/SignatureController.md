# SignatureController

> **檔案路徑**：`EEPWebClient.Core/Controllers/SignatureController.cs`（94 行）

## 用途

電子簽章功能，提供使用者手寫簽名圖片的上傳與下載。每位使用者一個簽名檔，以 PNG 格式儲存。

## URL 路由

預設 MVC 路由：`/Signature/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | GET | `/Signature` | 無 | FileResult / NotFound | 下載目前使用者的簽名圖片 |
| `Index` | POST | `/Signature` | `IFormCollection`（含 `value`：Base64 字串） | string / JSON | 上傳（儲存）簽名圖片 |

## 關鍵邏輯

### 簽名下載（`GetSignature`）

1. 從 Session 取得 `clientInfo`
2. 簽名檔路徑：`design/signature/{User}.png`
3. 讀取檔案 byte[] 並回傳；檔案不存在則回傳 `NotFound`

### 簽名上傳（`UploadSignature`）

1. 驗證 Session 登入狀態
2. 自動建立 `design/signature/` 目錄（若不存在）
3. 從表單的 `value` 欄位取得 Base64 編碼的圖片資料
4. 解碼後以 `FileMode.Create` 寫入 `design/signature/{User}.png`

## 備註

- 每位使用者僅保留一個簽名檔，新上傳會覆蓋舊簽名
- 前端以 Canvas 繪製簽名後轉為 Base64 傳送，非傳統檔案上傳
- 簽名檔固定為 PNG 格式
- `GetSignature` 方法接收 `FileType` 參數但實際未使用（呼叫時傳入 `FileType.Files`）
