# GlobalFileProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/GlobalFileProvider.cs` |
| 行數 | 56 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

全域檔案管理 Provider，提供讀取與儲存 bootstrap 目錄下的設定檔案（如 `_custom.css`、`_custom.js`）。

## mode 分派

| mode | 說明 |
|------|------|
| `get` | 讀取 `design/bootstrap/{solution}/_{fileName}.{fileType}` |
| `save` | 寫入同路徑檔案 |

## 備註

- 檔名前綴底線 `_`，如 `_custom.css`
