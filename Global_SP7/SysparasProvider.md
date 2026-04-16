# SysparasProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/SysparasProvider.cs` |
| 行數 | 28 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `UserDesignProvider` |

## 用途

系統參數管理 Provider，提供系統參數的儲存功能。

## mode 分派

| mode | 說明 |
|------|------|
| `save` | 儲存系統參數（columnName、title、values） |

## 備註

- `SaveSysParas` 方法繼承自 `UserDesignProvider`
- 目前僅實作 `save` 模式
