# UserMenuProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/UserMenuProvider.cs` |
| 行數 | 85 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |

## 用途

使用者選單檢查 Provider。**整個檔案已被註解掉**，目前不產生任何作用。

## 原設計功能

| mode | 說明 |
|------|------|
| `check` | 檢查指定 id 的選單在各類型（table / trans / 其他）下是否存在 |
| `checkCapacity` | 檢查容量（永遠回傳 true） |

## 備註

- 全部程式碼已註解，繼承自 `UserDesignProvider`
- 檢查邏輯包含：表格是否存在（`CheckTableExist`）、TRS 是否存在（查 SYS_TRSFILES）、JSON 檔是否存在
