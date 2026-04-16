# HttpContextExtensions

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Extension/HttpContextExtensions.cs` |
| 行數 | 29 行 |
| 命名空間 | `Microsoft.AspNetCore.Http` |

## 用途

`HttpContext` 的擴充方法，取得當前請求的語系。

## 擴充方法

| 方法 | 說明 |
|------|------|
| `GetLocale()` | 從 `Accept-Language` Header 取得語系名稱（取第一個語系） |

## 備註

- 優先使用瀏覽器 `Accept-Language` Header，忽略 `IRequestCultureFeature` 的結果
