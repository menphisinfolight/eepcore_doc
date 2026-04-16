# IfElseActivity（條件判斷活動）

> 設計端：`EEPFlowTools/Activities/IfElseActivity.cs`（19 行）
> 引擎端：`EEP.Flow/Activities/IfElseActivity.cs`（126 行）

## 用途

條件判斷活動，包含多個 `IfElseBranchActivity` 分支，依據條件運算式或伺服器方法判定走哪一條分支。類似 if / else if / else 邏輯。

## 繼承

```
Activity → CompositedActivity → IfElseActivity
                              → IfElseBranchActivity（繼承 SequenceActivity）
```

## 設計介面屬性

### IfElseActivity

無額外屬性。

### IfElseBranchActivity

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `Expression` | string | — | 條件運算式 |
| `Function` | string | `ServerMethodEditor` | 伺服器方法名稱 |

## 引擎執行邏輯

### Children（覆寫）

核心邏輯：僅回傳**符合條件的那一個分支**（而非所有子活動）。

1. 依序對每個 `IfElseBranchActivity` 呼叫 `Invoke()`
2. 第一個回傳 `true` 的分支即為選中分支
3. 若所有分支都不符合，找 `IsEmpty`（Expression 和 Function 皆為空）的分支作為 else
4. 若連 else 分支都沒有，拋出 `NotSupportedException`

### IfElseBranchActivity.Invoke()

- 若有 `Expression`：呼叫 `Instance.JudgeExpression(Expression)` 判定
- 若有 `Function`：呼叫 `Instance.Invoke(Function)`，比較結果是否為 "True"

### Reset()

清除 `trueActivities` 快取，並重置所有子活動。

### And 屬性

固定為 `false`（Or 模式），因為只會走一條分支。

## 備註

- `trueActivities` 為延遲計算並快取，首次存取 `Children` 時才評估條件
- `IfElseBranchActivity` 若 Expression 和 Function 都為空，視為 else 分支（`IsEmpty = true`）
