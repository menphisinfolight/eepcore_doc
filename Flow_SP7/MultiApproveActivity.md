# MultiApproveActivity（多人會簽活動）

> 設計端：`EEPFlowTools/Activities/MultiApproveActivity.cs`（47 行）
> 引擎端：`EEP.Flow/Activities/MultiApproveActivity.cs`（173 行）

## 用途

多人會簽活動，將簽核任務同時派發給多位使用者或角色，以**並行**方式執行。根據 `Mode` 設定，可選擇全部簽完（And）或任一人簽完（Or）即通過，並可透過 `Rate` 設定通過百分比。

## 繼承

```
Activity → CompositedActivity → ParallelActivity → MultiApproveActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| `SendTo` | string | `ValueOptionEditor("netflow", "multiApproveActivity")` | — | 接收者（支援 RefRole / RefUser） |
| `Mode` | ParallelMode | 下拉 | And | 並行模式：And（全簽）/ Or（任一簽） |
| `Rate` | int | `NumberboxEditor(0~100)` | 0 | 通過比例（Mode=And 時，< 100 表示達比例即通過） |
| `Parameters` | string | — | — | 傳遞給子活動的參數 |
| `PlusMode` | PlusMode | 下拉 | Disabled | 加簽模式（Disabled / Enabled / Agent） |
| `ExpDay` | double | `NumberboxEditor(0, 小數2位)` | 0 | 逾時天數 |
| `CanEdit` | bool | `CheckboxEditor` | true | 是否支援編輯 |
| `CanReturn` | bool | `CheckboxEditor` | true | 是否支援退回 |
| `CanAgent` | bool | `CheckboxEditor` | true | 是否支援代理 |
| `CanReject` | bool | — | false | 是否支援作廢 |

設計端另定義 `RefRole` / `RefUser` 內部類別（實作 `IValueType`），用於 `SendTo` 的動態參照。

## 引擎執行邏輯

### Start()

1. 若流程非退回/收回狀態，動態建立子活動：
   - 解析 `SendTo`：透過 Regex 匹配 `RefRole['欄位']` 或 `RefUser['欄位']`，從 Instance 取得實際角色/使用者清單
   - 解析 `Role`、`User` 屬性（逗號或分號分隔）
   - 為每個角色/使用者建立 `StandActivity` 子活動，ID 格式為 `{ID}_role_{i}` / `{ID}_user_{i}`
2. 子活動繼承父活動的 `CanReturn`、`CanEdit`、`CanReject`、`CanAgent`、`PlusMode`、`ExpDay` 等屬性
3. 呼叫 `base.Start()`（ParallelActivity）同時啟動所有子活動

### 狀態判定（繼承自 ParallelActivity）

- **And 模式（Rate=100）**：所有子活動完成 → Executed
- **And 模式（Rate<100）**：已完成子活動數 / 總數 >= Rate% → Executed
- **Or 模式**：任一子活動完成 → Executed

## 備註

- 與 `SeqApproveActivity` 的差異：本元件為並行簽核，所有人同時收到待辦；SeqApprove 為循序簽核，依序一人一人簽
- 引擎端額外有 `Role`、`User` 屬性（`[FlowEditor]`），設計端則統一透過 `SendTo` 設定
