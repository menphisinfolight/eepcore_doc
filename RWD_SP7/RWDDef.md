# RWDDef

> `EEPRWDTools.Core/RWDDef.cs` — 129 行

## 用途

**RWD 版本定義與資源管理**（RWD Definition）。

RWDDef 是一個抽象類別，定義了不同 Bootstrap 版本所需的 JavaScript、CSS 及 HTML class 名稱對應。透過 Singleton 模式（`Current`）依組態自動選擇 Bootstrap 3 或 Bootstrap 5 的實作。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Current` | RWDDef（static） | Singleton 實例，依 `RWD_VER` 組態決定版本 |
| `RWD_Ver` | string（static） | 從 `appsettings.json` 讀取 `RWD_VER`，預設 `"3.3.7"` |
| `Scripts` | string[]（abstract） | 該版本需載入的 JS 檔案路徑 |
| `StyleSheets` | string[]（abstract） | 該版本需載入的 CSS 檔案路徑 |
| `CustomScript` | string（abstract） | 版本專屬的自訂 JS 路徑 |
| `ItemDef` | Dictionary\<string, string\>（abstract） | HTML class 名稱對應表 |

## 版本實作

### RWD3Def（Bootstrap 3.3.7）

| Key | CSS Class |
|-----|-----------|
| panel | `panel` |
| panelHeader | `panel-heading` |
| panelBody | `panel-body` |
| panelFooter | `modal-footer` |
| modalHeader | `modal-header bg-primary` |
| formGroup | `form-group` |
| dataToggle | `data-toggle` |
| dataBackdrop | `data-backdrop` |
| in | `in` |

### RWD5Def（Bootstrap 5.3.3）

| Key | CSS Class |
|-----|-----------|
| panel | `card` |
| panelHeader | `panel-heading card-header` |
| panelBody | `card-body` |
| panelFooter | `card-footer text-end` |
| modalHeader | `modal-header` |
| formGroup | `row` |
| dataToggle | `data-bs-toggle` |
| dataBackdrop | `data-bs-backdrop` |
| in | `show in` |

## 版本選擇邏輯

```
appsettings.json["RWD_VER"]
  → 以 "5" 開頭 → RWD5Def（Bootstrap 5.3.3）
  → 其他或未設定 → RWD3Def（Bootstrap 3.3.7，預設）
```

## 備註

- `Configuration` 屬性以 Lazy 方式讀取 `appsettings.json`，僅讀取一次後快取。
- `Current` 也是 Lazy Singleton，首次存取時建立，之後不再變更。
- `ItemDef` 在控制項 Render 時廣泛使用（如 `RWDControl.ItemDef`），讓同一套程式碼能相容 Bootstrap 3 和 5 的 class 差異。
- Bootstrap 5 將 `data-toggle` 改為 `data-bs-toggle`、`panel` 改為 `card`，此類別集中處理了這些遷移差異。
