# Enum.cs（設計端列舉定義）

> 設計端：`EEPFlowTools/Enum.cs`（19 行）

## 用途

定義流程設計工具中使用的列舉型別。

## PlusMode（加簽模式）

| 值 | 說明 |
|----|------|
| `Disabled` | 停用加簽 |
| `Enabled` | 啟用加簽 |
| `Agent` | 代理加簽 |

## ParallelMode（並行模式）

| 值 | 說明 |
|----|------|
| `And` | 全部完成才算完成 |
| `Or` | 任一完成即算完成 |

## 備註

- 引擎端在 `ParallelActivity.cs` 中也定義了 `ParallelMode` 列舉
- 引擎端的 `PlusMode` 定義位於 `EEP.Flow/Activities/StandActivity.cs`
- 這些列舉同時影響設計介面的下拉選項和引擎端的行為判定
