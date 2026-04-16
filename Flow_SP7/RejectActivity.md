# RejectActivity（作廢活動）

> 設計端：`EEPFlowTools/Activities/RejectActivity.cs`（18 行）
> 引擎端：`EEP.Flow/Activities/RejectActivity.cs`（73 行）

## 用途

作廢活動，屬於系統活動。當流程走到此節點時，自動作廢整個流程。可設定作廢時通知的對象。

## 繼承

```
Activity → SystemActivity → RejectActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `NotifyRole` | string | `FlowRoleEditor` | 作廢通知角色 |
| `NotifyUser` | string | `FlowUserEditor` | 作廢通知使用者 |
| `NotifySendTo` | string | `ValueOptionEditor("netflow", "notifyActivity")` | 動態通知接收者 |
| `Remark` | string | — | 作廢意見 |

## 引擎執行邏輯

### Invoke()

呼叫 `Instance.Reject(ID)` 執行作廢操作。

### Reject()

產生作廢通知：
1. 若 `NotifyRole`、`NotifyUser`、`NotifySendTo` 皆為空 → 回傳空清單（不通知）
2. 否則建立 `NotifyActivity`，Text 為 `{ActivityText}_Reject`，呼叫其 `Invoke()` 發送通知

### SupportReject

覆寫為 `true`。

## 備註

- 繼承自 `SystemActivity`，`User` 固定為 "System"
- 此為「自動作廢」，通常放在條件分支中；使用者手動作廢則透過 RootActivity 或 StandActivity 的作廢功能
- 通知機制與 NotifyActivity 共用，支援角色、使用者、動態接收者三種方式
