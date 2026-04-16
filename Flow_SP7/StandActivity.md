# StandActivity（標準簽核活動）

## 檔案資訊

| 項目 | 路徑 | 行數 |
|------|------|------|
| 設計端 | `EEPFlowTools/Activities/StandActivity.cs` | 71 行 |
| 引擎端 | `EEP.Flow/Activities/StandActivity.cs` | 789 行 |

繼承：`Activity`（引擎端直接繼承抽象基底類別 `EEP.Flow.Activities.Activity`）

---

## 用途

StandActivity 是 EEP Flow 流程引擎中**最核心、最常用的活動元件**，代表一個「標準簽核關卡」。每個 StandActivity 對應流程中的一個人工審核步驟，指定由某個角色（Role）或使用者（User）進行簽核。

主要功能包括：
- **指定簽核人**：透過 Role（群組）或 User（使用者）直接指派，或透過 SendTo 動態解析
- **加簽（Plus）**：支援在簽核過程中臨時加入額外簽核人
- **轉簽（Transfer）**：將待辦轉交給其他人
- **退回（Return/Reset）**：退回至前一關卡或指定關卡
- **作廢（Reject）**：作廢整個流程並發送通知
- **通知（Notify）**：對指定人員發送通知
- **代理（Agent）**：支援代理人機制
- **逾時控制**：透過 ExpDay 設定逾期天數

---

## JSON 設定範例

流程設計器中的 StandActivity 節點設定（XML 屬性格式）：

```json
{
  "type": "StandActivity",
  "ID": "activity1",
  "Text": "主管審核",
  "Role": "MGR001",
  "User": "",
  "SendTo": "Manager",
  "Parameters": "",
  "AllowEmpty": false,
  "PlusMode": "Disabled",
  "ExpDay": 3,
  "CanEdit": true,
  "CanPrint": false,
  "CanReturn": true,
  "CanAgent": true,
  "CanReject": false
}
```

### SendTo 動態解析範例

SendTo 支援多種動態指定簽核人的語法：

| SendTo 值 | 說明 |
|-----------|------|
| `Applicant` | 申請人本人 |
| `Manager` | 目前使用者的主管 |
| `ApplicantManager` | 申請人的主管 |
| `RefRole['欄位名']` | 從表單欄位取得角色編號 |
| `RefUser['欄位名']` | 從表單欄位取得使用者編號 |
| `RefManager['欄位名']` | 從表單欄位取得角色，再取該角色的主管 |

---

## 設計介面屬性

以下屬性來自 `EEPFlowTools/Activities/StandActivity.cs`，含設計器 Attribute 標記：

| 屬性 | 型別 | 預設值 | 設計器 Attribute | 說明 |
|------|------|--------|------------------|------|
| Role | `string` | — | `[FlowRoleEditor]` | 群組（角色）編號 |
| User | `string` | — | `[FlowUserEditor]` | 使用者編號 |
| SendTo | `string` | — | `[ValueOptionEditor("netflow", "standActivity", true)]` | 動態接收者設定（含 TYPE 子選項） |
| Parameters | `string` | — | （無） | 自訂參數 |
| AllowEmpty | `bool` | `false` | （無） | 允許接收者為空白（空白時自動跳過） |
| PlusMode | `PlusMode` | `Disabled` | （無） | 加簽模式 |
| ExpDay | `double` | `0` | `[NumberboxEditor(0, true, 0, 2)]` | 逾時天數（0 = 不限） |
| CanEdit | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許編輯表單 |
| CanPrint | `bool` | `false` | （無） | 是否允許列印 |
| CanReturn | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許退回 |
| CanAgent | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許代理 |
| CanReject | `bool` | `false` | （無） | 是否允許作廢 |

### SendTo 內部 IValueType 子類別

設計端定義了以下 IValueType 實作，對應 SendTo 的選項值：

| 類別 | Field 子屬性 | 說明 |
|------|-------------|------|
| `Applicant` | — | 申請人 |
| `Manager` | — | 主管 |
| `ApplicantManager` | — | 申請人主管 |
| `RefRole` | `Field` | 參照欄位取角色 |
| `RefUser` | `Field` | 參照欄位取使用者 |
| `RefManager` | `Field` | 參照欄位取主管 |

### PlusMode 列舉

| 值 | 說明 |
|----|------|
| `Disabled` | 不支援加簽 |
| `Enabled` | 支援加簽 |
| `Agent` | 加簽結束後自動審核（代理加簽） |

---

## 引擎執行邏輯

### 類別結構

引擎端 `StandActivity` 繼承自抽象類別 `Activity`，除了設計端的屬性外，還增加了以下引擎專用屬性：

| 屬性 | 型別 | 說明 |
|------|------|------|
| ReturnTo | `string` | 退回目標活動（`[FlowEditor("activity")]`） |
| Duration | `int` | 持續時間（天數）（`[FlowEditor]`） |
| OnSubmit | `string` | 提交後觸發的方法名稱（`[FlowEditor]`） |
| SubmitResult | `string` | 提交結果欄位（`[FlowEditor]`） |
| ParentActivityID | `string` | 父活動 ID（加簽時使用） |

### Start() — 啟動活動（第 605-689 行）

啟動流程的核心方法，邏輯如下：

1. **動態解析 SendTo**（非退回/收回狀態時）：
   - 用正規表達式 `(\w+)(\[\'(\w+)'\])?` 解析 SendTo 字串
   - 依據 type 值（Applicant / Manager / ApplicantManager / RefRole / RefUser / RefManager）動態設定 Role 或 User
2. **空白處理**：
   - 若 Role 和 User 都為空且 `AllowEmpty = true`，直接標記為 Executed 並跳至下一個活動
   - 若 `AllowEmpty = false`，拋出例外
3. **建立待辦**：
   - 設定狀態為 `Waiting`，記錄開始時間
   - 呼叫 `InsertTodo()` 寫入待辦記錄

### Execute() — 完成簽核（第 296-311 行）

1. 驗證狀態必須為 `Waiting`
2. 呼叫 `CheckPlus()` 確認沒有未完成的加簽
3. 設定狀態為 `Executed`
4. 刪除待辦記錄（`DeleteTodo()`）
5. 若有設定 `OnSubmit`，觸發對應方法

### Plus() — 加簽（第 369-385 行）

1. 驗證狀態為 Waiting
2. 若 `isAgent = true`，將 PlusMode 設為 Agent
3. 對每個 user/role 建立子 StandActivity 並啟動

### PlusApprove() — 完成加簽（第 392-410 行）

1. 從資料庫取得加簽活動記錄
2. 執行該加簽活動的 Execute() 與 Log()
3. 更新參數，回傳剩餘等待中的加簽活動

### PlusReturn() — 退回加簽（第 417-434 行）

1. 取得加簽活動，呼叫 Reset() 重置
2. 刪除所有加簽記錄

### PlusTransfer() — 轉簽加簽（第 444-466 行）

1. 完成原加簽活動
2. 建立新的加簽活動給新的 users/roles

### Transfer() — 轉簽（第 474-500 行）

1. 若有 plusID，轉交加簽活動
2. 否則記錄歷史，更新 Role 或 User，更新待辦

### Reject() — 作廢（第 507-521 行）

- 若流程根活動有設定 `RejectSendTo`，建立 NotifyActivity 發送作廢通知

### Reset() — 重置（第 568-590 行）

1. 可選檢查狀態與加簽
2. 若狀態為 Waiting，刪除待辦和加簽記錄
3. 重置狀態為 Initialized

### Notify() — 通知（第 327-360 行）

兩個多載：
- `Notify(users, roles)`：對指定的 users/roles 各建立 NotifyActivity 並啟動
- `Notify(textSuffix, notifyType)`：對活動本身的 User/Role 發送通知

### IsWaiting — 是否需要等待（第 125-138 行）

- 若 `AllowEmpty = false`，一律需要等待
- 若 `AllowEmpty = true`，只在 Role 或 User 非空時需要等待

### ExpDatetime — 逾時時間計算（第 235-258 行）

取流程根活動逾時和本活動 ExpDay 兩者的**最小值**作為最終逾時時間。

### IsAgented / IsParAgented — 代理判斷（第 750-768 / 263-290 行）

- `IsAgented`：判斷目前使用者是否非原始簽核人（即代理中）
- `IsParAgented`：判斷是否為平行代理（透過 AgentInfo 查詢）

### 資料庫操作

透過 `FlowTodoHelper` 和 `FlowHistoryHelper` 操作：

| 方法 | 說明 |
|------|------|
| `InsertTodo()` | 新增待辦 |
| `DeleteTodo()` | 刪除待辦 |
| `UpdateTodo()` | 更新待辦（轉簽用） |
| `DeletePlus()` | 刪除加簽記錄 |
| `UpdateParameter()` | 更新活動參數 |
| `Log()` | 寫入歷史記錄 |

---

## 與官方文件差異

### 官方列出 15 項屬性

官方文件：Title, Role, User, SendTo (TYPE/Field), Parameters, AllowEmpty, PlusMode, ExpDay, CanEdit, CanPrint, CanReturn, CanAgent, CanReject

### 原始碼有但官方未列出的屬性

| 屬性 | 來源 | 說明 |
|------|------|------|
| **ReturnTo** | 引擎端 | 退回目標活動，有 `[FlowEditor("activity")]` 標記 |
| **Duration** | 引擎端 | 持續時間（天數），有 `[FlowEditor]` 標記 |
| **OnSubmit** | 引擎端 | 提交後事件方法名稱，有 `[FlowEditor]` 標記 |
| **SubmitResult** | 引擎端 | 提交結果欄位，有 `[FlowEditor]` 標記 |

> 這些屬性在引擎端有 `[FlowEditor]` 標記，表示在流程設計器中可設定，但官方文件未記載。

### 官方有但原始碼可能對應不同的項目

- **Title**：官方列出的 Title 在原始碼中對應的是基底類別 `Activity.Text` 屬性（XML 屬性名 `Text`），設計端的 Activity 基底類別為空類別，Title 可能由設計器 UI 層面處理（對應節點的顯示名稱）。

### SendTo 的 RefManager 子類別

設計端和引擎端都支援 `RefManager`，但官方文件的 SendTo 僅列出 TYPE/Field 兩個子屬性，未詳細列出所有可用的 TYPE 值。原始碼支援 6 種：Applicant、Manager、ApplicantManager、RefRole、RefUser、RefManager。

---

## 備註

1. **StandActivity 是最重要的活動元件**，幾乎所有需要人工簽核的關卡都使用它。`ApproveActivity` 的子關卡 `ApproveBranchActivity` 也繼承自 StandActivity。

2. **加簽的三種模式**：
   - `Disabled`：不允許加簽
   - `Enabled`：允許加簽，加簽完成後仍需原簽核人審核
   - `Agent`：加簽完成後自動通過，原簽核人不需再審核

3. **AllowEmpty 的妙用**：當 SendTo 動態解析後 Role/User 為空時，若 `AllowEmpty = true`，活動會自動跳過（直接 Executed）並繼續執行下一個活動。這在條件性簽核場景中很實用。

4. **退回與收回時不重新解析 SendTo**：Start() 中只在非 Return/Retake 狀態時才解析 SendTo，退回時沿用原有的 Role/User 值。

5. **逾時機制詳細說明**（見下方獨立章節）

6. **CheckPlus 驗證**：Execute() 和 Reset() 時會檢查是否還有未完成的加簽，若有則拋出 `flowPlusApproving` 例外，防止在加簽未完成時就結束活動。

---

## ⚠️ ExpDay 逾時機制詳細說明

### 基本概念

ExpDay 是 `double` 型別，單位為**日曆天**（Calendar Days），支援小數。

> **重要**：ExpDay 計算的是**日曆天**，不是工作天。不會排除週末、假日，也**不參考任何行事曆或假日表**。

### 設定範例

| ExpDay 值 | 意義 | 換算 |
|-----------|------|------|
| `0` | 不限時（預設） | 永不逾時 |
| `1` | 1 天 | 24 小時 |
| `0.5` | 半天 | 12 小時 |
| `0.25` | 6 小時 | — |
| `0.167` | 約 4 小時 | 0.167 × 24 = 4.008 小時 |
| `0.0417` | 約 1 小時 | 0.0417 × 24 ≈ 1 小時 |
| `3` | 3 天 | 72 小時 |
| `7` | 7 天 | 168 小時 |

### 計算公式

```csharp
// 原始碼：StandActivity.cs
public DateTime? ExpDatetime
{
    get
    {
        var dateTimes = new List<DateTime> { };
        
        // 1. 流程根活動的全域逾時
        var rootExpDatetime = Instance.RootActivity.ExpDatetime;
        if (rootExpDatetime.HasValue)
        {
            dateTimes.Add(rootExpDatetime.Value);
        }
        
        // 2. 本活動的逾時（活動開始時間 + ExpDay 天）
        if (ExpDay > 0)
        {
            dateTimes.Add(StartDateTime.AddDays(ExpDay));
        }
        
        // 3. 取最小值（最早到期的那個）
        if (dateTimes.Count > 0)
        {
            return dateTimes.Min();
        }
        return null;  // 不限時
    }
}
```

**關鍵**：`DateTime.AddDays(double)` 是 .NET 原生方法，直接加上日曆天的時間量，**不考慮任何假日邏輯**。

### 逾時判斷（前端）

```javascript
// bootstrap.infolight.nodeflow.js
rowStyler: function(index, row) {
    if (row.ExpDatetime) {
        var expDate = new Date(row.ExpDatetime);
        var now = new Date();
        if (expDate.getTime() < now.getTime()) {
            return 'color:#F00';  // 逾時顯示紅色
        }
    }
}
```

前端在待辦清單中，將逾時的項目**文字變紅色**顯示。

### 兩層逾時機制

| 層級 | 設定位置 | 計算方式 | 說明 |
|------|---------|---------|------|
| **流程級** | RootActivity.ExpDay | 流程啟動時間 + ExpDay | 整個流程的最終期限 |
| **活動級** | StandActivity.ExpDay | 活動開始時間 + ExpDay | 單一關卡的期限 |

最終逾時 = `Min(流程級, 活動級)`，確保單一關卡的期限不會超過流程整體期限。

### 常見問題

#### Q: 如何設定 4 小時逾時？
A: `ExpDay = 0.17`（0.17 × 24 = 4.08 小時）。因為設計介面是 2 位小數的數字框，實際精度到分鐘級。

#### Q: 是否排除週末和假日？
A: **否**。ExpDay 是純日曆天計算（`DateTime.AddDays`），不參考任何行事曆、假日表或工作日設定。週五下午設定 1 天逾時 = 週六下午到期。

#### Q: 逾時後會自動簽核嗎？
A: **不會自動簽核**。逾時只是在前端待辦清單中**顯示紅色**，不會觸發任何自動動作。若需要逾時自動處理，需搭配 `RootActivity.DelaySendMail`（逾時自動發郵件通知）或自訂排程。

#### Q: 逾時精度是多少？
A: 設計介面的 NumberboxEditor 允許 2 位小數，最小有效設定約 `0.01` 天 = 14.4 分鐘。`DateTime.AddDays` 本身支援到 tick 級精度。
