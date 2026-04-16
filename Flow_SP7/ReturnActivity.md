# ReturnActivity（退回活動）

> 設計端：`EEPFlowTools/Activities/ReturnActivity.cs`（14 行）
> 引擎端：`EEP.Flow/Activities/ReturnActivity.cs`（54 行）

## 用途

退回活動，屬於系統活動。當流程走到此節點時，自動將流程退回到指定的活動。常用於條件不符時自動退回。

## 繼承

```
Activity → SystemActivity → ReturnActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `ReturnTo` | string | `ActivityEditor` | 退回目標活動（從流程圖選取） |
| `Remark` | string | — | 退回意見 |

## 引擎執行邏輯

### Start()

驗證 `ReturnTo` 是否有設定，若為空則拋出 `NotSupportedException`。然後呼叫 `base.Start()` 回傳自身。

### Invoke()

呼叫 `Instance.Return(ID, ReturnTo)` 執行退回操作，將流程回退到指定活動。

### SupportReturn

覆寫為 `true`。

## 備註

- 繼承自 `SystemActivity`，`User` 固定為 "System"，`IsWaiting = false`
- `ReturnTo` 為必填，未設定會在 Start 時拋出例外
- 此為「自動退回」，與使用者手動按退回按鈕不同（手動退回由 StandActivity 等處理）
