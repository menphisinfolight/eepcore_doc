# SolutionProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/SolutionProvider.cs` |
| 行數 | 77 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

方案（Solution）管理 Provider，提供方案資料的 CRUD 與切換功能。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 查詢方案資料（通用 GetDataset） |
| `save` | 儲存方案資料（通用 UpdateDataset，支援 XSS 驗證開關） |
| `get` | 取得所有方案清單（MENUITEMTYPE），標記當前方案 |
| `set` | 切換當前方案（更新 Session 與 Cookie） |

## 備註

- `set` 會將方案寫入 `devsolution` Cookie（有效期 7 天）
- `save` 讀取 `validateXss` 設定決定是否停用 XSS 驗證
