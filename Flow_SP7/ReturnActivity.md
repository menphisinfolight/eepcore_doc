# ReturnActivity（退回活動）

> 設計端：`EEPFlowTools/Activities/ReturnActivity.cs`（14 行）
> 引擎端：`EEP.Flow/Activities/ReturnActivity.cs`（54 行）
> 退回邏輯：`EEP.Flow/Instance.cs` — `Return()` 方法 + `Activity.GetPreviousActivity()`

## 用途

退回活動，屬於系統活動。當流程走到此節點時，自動將流程退回到指定的活動。常用於條件不符時自動退回。

## 繼承

```text
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

---

## ⚠️ 退回機制完整分析

### 哪些活動可以被退回？

退回能力由兩個條件共同決定：

#### 1. SupportReturn — 當前活動是否支援退回操作

| 活動類型 | SupportReturn | 說明 |
|----------|:------------:|------|
| **StandActivity** | `CanReturn`（設計師設定） | 預設 true，可在設計介面關閉 |
| **ReturnActivity** | `true` | 固定支援 |
| **CompositedActivity**（ParallelActivity、IfElseActivity） | `false` | 容器活動不支援退回 |
| **ContinuedActivity** 基底 | `false` | 預設不支援，由子類覆寫 |
| **SystemActivity** 基底 | `false` | 系統活動預設不支援 |
| **DetailFlowActivity** | `false` | 會簽明細不支援退回 |
| **DetailResultActivity** | `false` | 會簽結果不支援退回 |

> **結論**：只有 **StandActivity**（含其子類 ApproveBranchActivity）和 **ReturnActivity** 可以執行退回。

#### 2. GetPreviousActivity — 能否找到退回目標

即使 SupportReturn = true，還需要 `GetPreviousActivity(returnTo)` 能成功找到目標活動。找不到會拋出 `NotSupportedException`。

### GetPreviousActivity 退回搜尋邏輯

```csharp
public virtual Activity GetPreviousActivity(string returnTo)
{
    var parent = GetParentActivity();  // 取得父容器
    if (parent != null)
    {
        var index = parent.Children.IndexOf(this);  // 在父容器中的位置
        
        if (parent.Sequence && index > 0)
        {
            // ✅ 父容器是循序（Sequence=true）且不是第一個子活動
            var previousActivity = parent.Children[index - 1];  // 取前一個活動
            previousActivity.Reset();
            
            if (!previousActivity.IsWaiting)
            {
                // 前一個不是等待型活動（如 ValidateActivity），繼續往前找
                return previousActivity.GetPreviousActivity(returnTo);
            }
            else if (!string.IsNullOrEmpty(returnTo) 
                     && previousActivity.ID != returnTo)
            {
                // 指定了退回目標，但還沒找到，繼續往前找
                return previousActivity.GetPreviousActivity(returnTo);
            }
            else
            {
                // ✅ 找到了！回傳這個活動
                return previousActivity;
            }
        }
        else
        {
            // ❌ 父容器不是循序（如 ParallelActivity），或是第一個子活動
            // → 跳到父容器層級繼續往前找
            parent.Reset();
            return parent.GetPreviousActivity(returnTo);
        }
    }
    return null;  // ❌ 到頂了還找不到 → 退回失敗
}
```

### 什麼情況「不能退回」？

| 情況 | 原因 | 錯誤訊息 |
|------|------|---------|
| **當前活動的 CanReturn = false** | 設計師在 StandActivity 關閉了退回權限 | `ThrowNotSupportException("Return")` |
| **當前活動是 CompositedActivity** | 容器型活動（並行/條件）SupportReturn = false | 同上 |
| **當前活動是 DetailFlowActivity** | 會簽明細活動 SupportReturn = false | 同上 |
| **退回目標在並行分支中** | 並行容器 `Sequence = false`，GetPreviousActivity 會跳過整個並行區塊向上找 | 找不到分支內的活動 |
| **退回目標是第一個活動** | `index = 0`，無法再往前 → 跳到父容器繼續找，若父容器也是第一個則到頂 | `GetPreviousActivity` 回傳 null → 拋錯 |
| **退回目標不在同一條路徑上** | 指定了 returnTo 但遞迴搜尋到頂都沒匹配的 ID | 回傳 null → 拋錯 |

### 並行分支中的退回限制（重點）

```text
RootActivity
├── StandActivity A（關卡 A）
├── ParallelActivity          ← Sequence = false
│   ├── Branch 1
│   │   └── StandActivity B1
│   └── Branch 2
│       └── StandActivity B2
└── StandActivity C（關卡 C）
```

- **C 退回到 A**：✅ 可以。GetPreviousActivity 會跳過 ParallelActivity（Sequence=false），從 RootActivity 找到 A。
- **B1 退回到 A**：✅ 可以。B1 在 Branch 1 中 index=0 → 跳到 Branch 1 → Branch 1 在 ParallelActivity 中但 Sequence=false → 跳到 ParallelActivity → ParallelActivity 在 RootActivity 中 index=1 → 往前找到 A。
- **C 退回到 B1**：❌ **不行**。搜尋路徑是 C → ParallelActivity（Sequence=false，跳過）→ A。永遠不會進入 Branch 內部搜尋。
- **B1 退回到 B2**：❌ **不行**。B1 和 B2 在不同分支中，互相不可見。

### 循序容器 vs 非循序容器

| 容器類型 | Sequence | 退回行為 |
|---------|----------|---------|
| **RootActivity** | `true`（繼承 SequenceActivity） | 可以在子活動之間退回 |
| **SequenceActivity**（分支） | `true` | 分支內可退回 |
| **ParallelActivity** | `false` | 跳過整個並行區塊 |
| **IfElseActivity** | `false`（繼承 CompositedActivity） | 跳過整個條件區塊 |

### Instance.Return() 完整流程

```text
1. 設定 _status = InstanceStatus.Return
2. 取得當前活動 → 檢查 SupportReturn
3. Clone 當前活動（用於代理通知）
4. Reset 當前活動
5. 決定退回目標：
   - 有指定 returnTo → 使用指定值
   - 無指定 → 使用 StandActivity.ReturnTo 屬性
6. GetPreviousActivity(returnTo) 搜尋目標
   - 找到 → 啟動目標活動（Start）
   - 找不到 → 拋出例外
7. 記錄日誌（Log）
8. 觸發 OnReturn 事件
9. 若下一個是 SystemActivity → 自動執行
10. 若是代理簽核 → 發送 AgentReturn 通知
11. 儲存 Instance
```

## 退回 vs 取回（Retake）

| 項目 | Return（退回） | Retake（取回） |
|------|--------------|--------------|
| **發起者** | 當前簽核人 | 上一個簽核人（送出後反悔） |
| **方向** | 退到指定或前一個活動 | 取回到自己（前一個活動） |
| **SupportReturn 檢查** | ✅ 檢查 | ❌ 不檢查（已被註解） |
| **returnTo 參數** | 可指定 | 固定為 null（前一個） |

## 備註

- 繼承自 `SystemActivity`，`User` 固定為 "System"，`IsWaiting = false`
- `ReturnTo` 為必填，未設定會在 Start 時拋出例外
- 此為「自動退回」（流程定義中的 ReturnActivity 節點），與使用者手動按退回按鈕不同（手動退回由 StandActivity 的 CanReturn 控制）
- 退回時會 Reset 當前活動的狀態，清除待辦記錄
- 代理簽核（ParAgent）退回時會額外發送 AgentReturn 通知給代理人
