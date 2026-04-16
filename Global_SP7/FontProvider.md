# FontProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/FontProvider.cs` |
| 行數 | 85 |
| 繼承 | `BaseProvider` |

## 用途

管理系統字型設定，提供字型清單的讀取、儲存與重置功能。字型定義存放於 `wwwroot/json/jquery.infolight.font.json`。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `get` | 讀取字型 JSON 檔案內容並回傳 |
| `save` | 將前端傳來的字型資料覆寫至字型 JSON 檔案 |
| `reset` | 從預設字型檔（`jquery.infolight.font.default.json`）還原字型設定 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetFonts(system)` | 讀取 `wwwroot/json/jquery.infolight.font.json` 並回傳字串內容 |
| `SaveFonts(datas)` | 將資料以 UTF-8 寫入字型 JSON 檔案 |
| `ResetFonts()` | 從 `jquery.infolight.font.default.json` 讀取預設值並覆寫至主字型檔 |

## 備註

- 字型檔路徑固定為 `wwwroot/json/jquery.infolight.font.json`。
- 預設字型檔路徑為 `wwwroot/json/jquery.infolight.font.default.json`。
- `system` 參數目前未使用（有相關跨平台字型路徑的程式碼已被註解）。
- 例外訊息錯誤引用 `IndexProvider`（應為 `FontProvider`）。
