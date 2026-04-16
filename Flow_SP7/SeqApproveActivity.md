# SeqApproveActivity（循序會簽活動）

> 設計端：`EEPFlowTools/Activities/SeqApproveActivity.cs`（41 行）
> 引擎端：`EEP.Flow/Activities/SeqApproveActivity.cs`（151 行）

## 用途

循序會簽活動，將簽核任務依序派發給多位使用者或角色，前一人簽完後才輪到下一人。適用於需要按順序逐一審核的場景。

## 繼承

```
Activity → CompositedActivity → SequenceActivity → SeqApproveActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| `SendTo` | string | `ValueOptionEditor("netflow", "seqApproveActivity")` | — | 接收者（支援 RefRole / RefUser） |
| `Parameters` | string | — | — | 傳遞給子活動的參數 |
| `PlusMode` | PlusMode | 下拉 | Disabled | 加簽模式 |
| `ExpDay` | double | `NumberboxEditor(0, 小數2位)` | 0 | 逾時天數 |
| `CanEdit` | bool | `CheckboxEditor` | true | 是否支援編輯 |
| `CanReturn` | bool | `CheckboxEditor` | true | 是否支援退回 |
| `CanAgent` | bool | `CheckboxEditor` | true | 是否支援代理 |
| `CanReject` | bool | — | false | 是否支援作廢 |

設計端另定義 `RefRole` / `RefUser` 內部類別，用於 `SendTo` 的動態參照。

## 引擎執行邏輯

### Start()

1. 若流程非退回/收回狀態，動態建立子活動：
   - 解析 `SendTo`：與 MultiApproveActivity 相同邏輯，透過 Regex 匹配 `RefRole['欄位']` / `RefUser['欄位']`
   - 解析直接指定的角色/使用者
   - 為每個角色/使用者建立 `StandActivity` 子活動
2. 呼叫 `base.Start()`（SequenceActivity）—— 僅啟動第一個子活動

### 狀態判定（繼承自 SequenceActivity）

- SequenceActivity 為循序執行，`Sequence = true`
- 前一個子活動完成後才啟動下一個
- 所有子活動完成 → Executed

## 備註

- 與 `MultiApproveActivity` 的差異：本元件繼承 `SequenceActivity`（循序），Multi 繼承 `ParallelActivity`（並行）
- `SendTo` 的解析邏輯與 MultiApproveActivity 完全相同
