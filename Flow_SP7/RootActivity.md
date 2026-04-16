# RootActivity（根活動）

> 設計端：`EEPFlowTools/Activities/RootActivity.cs`（44 行）
> 引擎端：`EEP.Flow/Activities/RootActivity.cs`（320 行）

## 用途

根活動，每個流程定義的最外層容器，也是流程的起點。負責管理流程全域設定（通知方式、逾時、作廢處理等），並作為啟動流程的第一個簽核節點（經辦人送出）。

## 繼承

```
Activity → CompositedActivity → SequenceActivity → RootActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| `FlowName` | string | — | — | 流程名稱 |
| `OrgKind` | string | `SecurityEditor("orgKind", ...)` | — | 組織類別 |
| `Database` | string | `DatabaseEditor` | — | 資料庫 |
| `SendNotify` | bool | `CheckboxEditor` | true | 是否發送系統通知 |
| `SendPush` | bool | `CheckboxEditor` | true | 是否發送推播 |
| `SendLine` | bool | `CheckboxEditor` | true | 是否發送 LINE 通知 |
| `SkipSameUser` | bool | `CheckboxEditor` | true | 同使用者自動簽核 |
| `CanPrint` | bool | — | false | 是否支援列印 |
| `DelayAutoApprove` | bool | — | false | 逾時自動簽核 |
| `ExpDay` | double | `NumberboxEditor(0, 小數2位)` | 0 | 逾時天數 |
| `RejectFunction` | string | `ServerMethodEditor` | — | 作廢時呼叫的方法 |
| `RejectSendTo` | string | `ValueOptionEditor("netflow", "notifyActivity")` | — | 作廢時通知對象 |
| `SiteCode` | string | — | — | 公司別 |
| `DisNotifyGroups` | string | `FlowRoleEditor(true)` | — | 排除通知的群組 |

## 引擎執行邏輯

### Start()

1. 非退回/收回時，設定 `StartUser` 為目前使用者
2. 設定狀態為 Waiting
3. 記錄啟動時間 `_startDateTime`
4. 插入待辦事項（`InsertTodo()`）
5. 回傳自身（等待經辦人送出）

### Execute()

經辦人送出簽核：
1. 檢查狀態必須為 Waiting
2. 驗證目前使用者 = StartUser（非本人不可簽）
3. 設定 StartRole
4. 標記為 Executed，刪除待辦

### GetNextActivity()

回傳 Children 的第一個子活動。

### GetPreviousActivity()

若 `returnTo` 為空或等於自身 ID → 回傳自身；否則回傳 null。

### Reject()

根活動的作廢：
1. 驗證目前使用者 = StartUser（僅經辦人可作廢）
2. 若有設定 `RejectSendTo`，建立 NotifyActivity 發送通知

### Log()

呼叫 `FlowHistoryHelper.Insert()` 寫入流程歷程。

### ToObject()

輸出：`[ID, ActivityText, "", "", StartUser, StartUserName, 0, false]`

### 引擎端額外屬性

| 屬性 | 說明 |
|------|------|
| `StartText` | 啟動活動文本（覆寫 ActivityText） |
| `StartUser` | 啟動流程的使用者 |
| `StartRole` | 啟動流程的群組 |
| `StartDateTime` | 啟動時間 |
| `ExpDatetime` | 逾時時間（StartDateTime + ExpDay） |
| `DelaySendMail` | 逾時是否發送郵件 |

## 備註

- RootActivity 同時是容器（SequenceActivity）和簽核節點（經辦人送出），雙重角色
- `SkipSameUser` 影響下游活動：若下一位簽核者與經辦人相同，可自動跳過
- `SupportReject = true`、`SupportEdit = true`、`SupportPrint` 由 `CanPrint` 決定
- `InsertTodo()` / `DeleteTodo()` 操作 FLOWTODO 資料表
