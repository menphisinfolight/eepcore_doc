# Activity（活動基底類別）

> 引擎端：`EEP.Flow/Activities/Activity.cs`（1125 行）
> 無設計端對應（設計端的 Activity 基底類別在 EEPFlowTools 中為空殼）

## 用途

所有流程活動的抽象基底類別，定義了活動的核心屬性、狀態管理、流程導航及操作介面。同一檔案中也包含三個中間抽象類別：`CompositedActivity`、`SequenceActivity`、`ContinuedActivity`、`SystemActivity`。

## ActivityStatus 列舉

| 值 | 名稱 | 說明 |
|----|------|------|
| 0 | Initialized | 初始化（尚未啟動） |
| 1 | Waiting | 等待簽核 |
| 2 | Executed | 已完成 |

## Activity 核心屬性

| 屬性 | 型別 | 說明 |
|------|------|------|
| `ID` | string | 活動編號（XML 屬性） |
| `Text` | string | 活動文本（XML 屬性） |
| `ActivityText` | string（virtual） | 顯示文本，預設回傳 Text |
| `Instance` | Instance | 所屬流程實例（XmlIgnore） |
| `NotifyType` | string | 通知類型 |

## 抽象屬性（子類別必須實作）

| 屬性 | 型別 | 說明 |
|------|------|------|
| `Status` | ActivityStatus | 活動狀態 |
| `IsWaiting` | bool | 是否等待中 |
| `SupportReturn` | bool | 支援退回 |
| `SupportPlus` | bool | 支援加簽 |
| `SupportAgent` | bool | 支援代理 |
| `SupportReject` | bool | 支援作廢 |
| `SupportEdit` | bool | 支援編輯 |
| `SupportPrint` | bool | 支援列印 |
| `DurationDays` | int | 持續天數 |
| `IsParAgented` | bool（virtual） | 是否平行代理（預設 false） |

## 核心方法

| 方法 | 說明 |
|------|------|
| `Execute()` | 完成活動（簽核送出） |
| `Start()` | 啟動活動，回傳所有啟動的活動清單 |
| `Reset()` | 重置活動狀態 |
| `Log()` | 記錄活動歷程 |
| `Notify()` | 通知活動（指定使用者/角色） |
| `Plus()` | 加簽（新增簽核者） |
| `PlusApprove()` | 完成加簽 |
| `PlusReturn()` | 退回加簽 |
| `PlusTransfer()` | 轉簽加簽 |
| `Transfer()` | 轉簽 |
| `Reject()` | 作廢活動 |
| `ToObject()` | 輸出為 object 陣列 |
| `ToParameter()` | 輸出參數字串 |

## 流程導航方法

| 方法 | 說明 |
|------|------|
| `GetParentActivity()` | 取得父活動 |
| `GetPreviousActivity(returnTo)` | 取得上一個活動（支援退回到指定活動） |
| `GetPreviousActivities()` | 取得所有已執行的前置活動清單 |
| `GetNextActivity()` | 取得下一個活動 |
| `GetPlanStart()` | 取得計畫開始時間 |
| `Clone()` | 淺複製 |

## 中間抽象類別

### CompositedActivity（複合活動，第 460 行）

容器型活動，可包含子活動。

| 屬性/方法 | 說明 |
|-----------|------|
| `ChildActivities` | 子活動清單（序列化用） |
| `Children` | 子活動清單（執行用，自動設定 Instance） |
| `And` | 是否為 And 模式（預設 true） |
| `Sequence` | 是否循序執行（預設 false） |
| `IsInitialized` | 是否已初始化（abstract） |
| `Status` | And：全部完成=Executed；Or：任一完成=Executed |
| `Start()` | 啟動所有子活動（並行） |
| `GetActivityByID()` | 遞迴搜尋子活動 |
| `GetParentActivity()` | 遞迴搜尋父活動 |
| `GetWaitingActivities()` | 取得所有等待中的活動 |
| `GetActivities<T>()` | 取得指定型別的所有活動 |

所有 Support* 屬性預設為 false，DurationDays = 0。

### SequenceActivity（循序活動，第 779 行）

繼承 CompositedActivity，`Sequence = true`。Start() 僅啟動第一個子活動。

### ContinuedActivity（可繼續活動，第 819 行）

非容器型的自動活動，Start() 時呼叫 `Invoke()` 並自動繼續。

| 特性 | 說明 |
|------|------|
| `IsWaiting` | 固定 false（不需等待） |
| `Invoke()` | 抽象方法，子類別實作具體邏輯 |
| `Start()` | 呼叫 Invoke()，若無 SystemActivity 回傳則標記 Executed 並繼續 |

所有 Support* 屬性預設為 false。

### SystemActivity（系統活動，第 965 行）

系統自動執行的活動（退回、作廢等）。

| 特性 | 說明 |
|------|------|
| `User` | 固定 "System" |
| `Remark` | 意見文字 |
| `IsWaiting` | 固定 false |
| `Start()` | 回傳自身 |
| `Invoke()` | 抽象方法，回傳 DataSet |
| `Log()` | 寫入 FlowHistoryHelper |

所有 Support* 屬性預設為 false。

## XmlInclude 註冊

Activity 類別透過 `[XmlInclude]` 註冊所有子類別，支援 XML 序列化/反序列化：
StandActivity、MultiApproveActivity、SeqApproveActivity、ReturnActivity、RejectActivity、NotifyActivity、PrewarningActivity、ProcedureActivity、ValidateActivity、ApproveActivity、ApproveBranchActivity、ParallelActivity、ParallelBranchActivity、IfElseActivity、IfElseBranchActivity、SubFlowActivity、DetailFlowActivity。

## 備註

- 所有虛擬方法預設拋出 `NotSupportedException`，子類別需覆寫所需的方法
- `GetPreviousActivity()` 為退回流程的核心邏輯，會遞迴向前尋找可等待的活動
- `GetNextActivity()` 在父活動 Status = Executed 時（如 Or 模式），會清除其他分支的待辦
