# SessionExtensions

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Extension/SessionExtensions.cs` |
| 行數 | 48 行 |
| 命名空間 | `Microsoft.AspNetCore.Http` |

## 用途

`ISession` 的擴充方法集，提供泛型序列化存取及 ClientInfo / Captcha 的便利操作。

## 擴充方法

| 方法 | 說明 |
|------|------|
| `Set<T>(key, value)` | 以 JSON 序列化存入 Session |
| `Get<T>(key)` | 從 Session 取出並反序列化 |
| `SetClientInfo(clientInfo)` | 儲存 ClientInfo（key = "ClientInfo"） |
| `GetClientInfo()` | 取得 ClientInfo |
| `ClearClientInfo()` | 清除 ClientInfo |
| `SetCaptcha(value)` | 儲存驗證碼（key = `captcha:{sessionId}`） |
| `GetCaptcha()` | 取得驗證碼 |

## 備註

- 使用 `System.Text.Json.JsonSerializer` 進行序列化
