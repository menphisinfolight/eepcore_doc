# ParallelActivity（並行活動）

> 設計端：`EEPFlowTools/Activities/ParallelActivity.cs`（21 行）
> 引擎端：`EEP.Flow/Activities/ParallelActivity.cs`（70 行）

## 用途

並行活動容器，可包含多條 `ParallelBranchActivity` 分支同時執行。根據模式設定決定全部完成或任一完成即通過。

## 繼承

```
Activity → CompositedActivity → ParallelActivity
                              → ParallelBranchActivity（繼承 SequenceActivity）
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| `Mode` | ParallelMode | 下拉 | And | And = 全部完成；Or = 任一完成 |
| `Rate` | int | `NumberboxEditor(0~100)` | 100 | 通過比例（僅 And 模式且 < 100 時生效） |

`ParallelBranchActivity` 為分支子元件，設計端無額外屬性。

## 引擎執行邏輯

### And 屬性

由 `Mode` 決定：`Mode == ParallelMode.And` 時回傳 `true`。

### Status（狀態判定）

- 無子活動 → 已初始化則 Executed，否則 Initialized
- **And 模式 + Rate < 100**：已完成子活動數 * 100 / 總數 >= Rate → Executed
- **And 模式 + Rate = 100**：所有子活動 Executed → Executed
- **Or 模式**：任一子活動 Executed → Executed

### Start()（繼承自 CompositedActivity）

同時啟動所有子活動（Children.ForEach → Start），與 SequenceActivity 只啟動第一個不同。

## 備註

- `ParallelBranchActivity` 繼承 `SequenceActivity`，每個分支內的活動仍為循序執行
- 引擎端定義了 `ParallelMode` 列舉（And / Or），與設計端 `Enum.cs` 中的定義對應
- `MultiApproveActivity` 繼承自本類別
