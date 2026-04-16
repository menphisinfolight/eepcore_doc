# GlobalProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/GlobalProvider.cs` |
| 行數 | 144 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

全域設定管理 Provider，提供 `global.cfg` 的載入/儲存、參數管理、Logo 設定、地圖設定及自訂腳本載入等功能。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 載入全域設定（含 publicKey） |
| `getAbbyyInfo` | 取得 Abbyy 資訊（空實作） |
| `getMapType` | 取得地圖類型（空實作） |
| `save` | 儲存全域設定 |
| `loadParam` | 載入參數設定 |
| `saveParam` | 儲存參數設定 |
| `changeKey` | 變更金鑰（空實作） |
| `setLogo` | 設定 Logo（刪除 images 下的 png） |
| `resetLogo` | 重設 Logo（刪除 images 下的 png） |
| `testSSOUrl` | 測試 SSO URL（空實作） |

## 額外公開方法

| 方法 | 說明 |
|------|------|
| `GetMapScript()` | 根據地圖設定產生 JS（MAP_TYPE + MAP_KEY + 地圖腳本） |
| `GetCustomScript()` | 載入 RWDDef 自訂腳本內容 |

## 備註

- Logo 路徑：`design/images/{logoType}.png`
- 地圖腳本路徑：`wwwroot/js/infolight/jquery.infolight.map.js`
