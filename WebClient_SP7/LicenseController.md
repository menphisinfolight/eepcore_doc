# LicenseController

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPWebClient.Core/Controllers/LicenseController.cs` |
| 行數 | 155 |
| 命名空間 | `EEPWebClient.Core.Controllers` |
| 繼承 | `Controller` |

## 用途

授權管理控制器，負責顯示授權狀態、註冊序號與匯入授權金鑰檔案。同檔案包含 `Encrypt` 輔助類別，透過反射呼叫 `EEPAdapter.dll` 進行加解密操作。

## URL 路由

| HTTP 方法 | 路由 | 說明 |
|-----------|------|------|
| GET | `/License` | 顯示授權資訊頁面 |
| POST | `/License/Register` | 以序號 + 密碼註冊 |
| POST | `/License/RegisterFile` | 以授權金鑰檔案註冊 |

## Actions 表

| Action | 方法 | 特性標記 | 回傳型別 | 說明 |
|--------|------|----------|----------|------|
| `Index()` | GET | `[HttpGet, AddMessage]` | `IActionResult` | 載入授權資訊並顯示頁面 |
| `Register(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 以公司名稱、序號、密碼註冊 |
| `RegisterFile(IFormCollection)` | POST | `[HttpPost]` | `IActionResult` | 以金鑰檔案註冊 |

## 關鍵邏輯

### Index（授權資訊載入）

`Load()` 私有方法執行以下步驟：
1. 取得主機 MAC 位址並計算 MD5 雜湊值（作為機器識別碼 `macValue`）。
2. 呼叫 `Encrypt.checkKey()` 檢查是否已註冊。
3. 設定 ViewBag 屬性：
   - `isRegistered`：是否已註冊
   - `text`：顯示文字（如 `EEPCore(已註冊)`、`EEPCloud(已註冊至{日期})`）
   - `sn`：序號
   - `company`：公司名稱
   - `macValue`：機器識別碼

### Register（序號註冊）

- 接收 `company`、`sn`、`password` 表單參數。
- 呼叫 `Encrypt.registerKey()` 完成註冊。
- 成功回傳空字串，失敗回傳 `InnerException.Message`。

### RegisterFile（檔案註冊）

- 接收上傳的金鑰檔案與 `sn` 參數。
- 支援 `.eepkey2` 格式：額外附加 `id` 參數至 mdKey。
- 將檔案讀入 byte 陣列後呼叫 `Encrypt.loadKey()` 載入金鑰。

### Encrypt 輔助類別（第 97-152 行）

透過反射載入 `EEPAdapter.dll`，呼叫 `EEPAdapter.Encrypt` 類別的方法：

| 方法 | 說明 |
|------|------|
| `getMac()` | 取得主機 MAC 位址 |
| `getMD5(string)` | 計算 MD5 雜湊 |
| `checkKey()` | 檢查授權金鑰（回傳含 `d`/`s`/`c` 屬性的動態物件） |
| `registerKey(company, sn, password)` | 以序號密碼註冊 |
| `loadKey(sn, buffer)` | 以金鑰檔案註冊 |

## 備註

- `Encrypt` 類別使用反射載入 `EEPAdapter.dll`，該 DLL 位於執行目錄下，採延遲載入（lazy load）模式。
- 授權區分兩種產品：`EEPCore`（永久授權）與 `EEPCloud`（有效期限授權），由 `checkKey()` 回傳的 `d` 屬性（到期日）判斷。
- `.eepkey2` 為新版金鑰檔案格式，需額外的 `id` 參數。
