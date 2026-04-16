# NotifyActivity（知會通知活動）

> 設計端：`EEPFlowTools/Activities/NotifyActivity.cs`（54 行）
> 引擎端：`EEP.Flow/Activities/NotifyActivity.cs`（191 行）

## 用途

知會通知活動，用於在流程進行中向指定的角色或使用者發送通知訊息。與簽核活動不同，通知活動**不需要收件者進行簽核操作**，僅作為訊息傳達用途。繼承自 `ContinuedActivity`，執行後自動銜接後續活動。

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| Title | string | （繼承自 Activity） | — | 活動標題（顯示名稱） |
| Role | string | `FlowRoleEditor` | — | 群組編號（接收通知的角色） |
| User | string | `FlowUserEditor` | — | 使用者編號（接收通知的使用者） |
| SendTo | string | `ValueOptionEditor("netflow", "notifyActivity", true)` | — | 接收者類型（下拉選單） |
| SendTo TYPE（內部類別） | — | — | — | 見下方「SendTo 類型」說明 |
| SendTo Field | string | — | — | RefRole / RefUser 時的欄位參考名稱 |
| AllowEmpty | bool | — | `false` | 允許接收者為空白（不拋例外） |
| LogHistory | bool | — | `false` | 是否記錄歷程日誌 |
| CanPrint | bool | — | `false` | 是否支援列印 |

### SendTo 類型（IValueType 內部類別）

| 類型 | 說明 | 額外屬性 |
|------|------|----------|
| `Applicant` | 申請人（流程啟動者） | 無 |
| `Manager` | 當前簽核者的主管 | 無 |
| `ApplicantManager` | 申請人的主管 | 無 |
| `RefRole` | 參考欄位指定的角色 | `Field`：欄位名稱 |
| `RefUser` | 參考欄位指定的使用者 | `Field`：欄位名稱 |
| `AllUsers` | 所有曾參與流程的使用者 | 無 |

## 引擎執行邏輯

### Invoke() 主流程

```
1. 若 SendTo 有值，解析類型（正則：(\w+)(\['(\w+)'\])?）
   ├─ Applicant     → User = 流程啟動者
   ├─ Manager       → Role = 當前簽核者的上級主管
   ├─ ApplicantManager → Role = 申請人的上級主管
   ├─ RefRole       → 從欄位取得多個角色 → 各自建立 NotifyActivity → InsertNotify + InsertHistory → 回傳
   ├─ RefUser       → 從欄位取得多個使用者 → 各自建立 NotifyActivity → InsertNotify + InsertHistory → 回傳
   └─ AllUsers      → 查詢歷程明細取得所有參與者（含當前使用者）→ 各自建立 NotifyActivity → InsertNotify + InsertHistory → 回傳

2. 若 SendTo 為空、且 Role 與 User 皆為空：
   ├─ AllowEmpty = true  → 回傳空清單（跳過通知）
   └─ AllowEmpty = false → 拋出 NotSupportedException

3. 一般情況：建立單一 NotifyActivity → InsertNotify() + InsertHistory() → 回傳
```

### 關鍵方法

| 方法 | 說明 |
|------|------|
| `InsertNotify()` | 透過 `FlowNotifyHelper.Insert()` 寫入通知記錄 |
| `InsertHistory()` | 當 `LogHistory = true` 時，透過 `FlowHistoryHelper.Insert()` 寫入歷程 |
| `CreateNotifyActivity()` | 以 `MemberwiseClone` 複製自身，替換 ID / User / Role，用於多人通知場景 |
| `ToObject()` | 輸出為 `object[]`：`{ ID, ActivityText, Role, GroupName, User, UserName, 0, false }` |

### 多人通知機制

RefRole、RefUser、AllUsers 三種類型會產生**多個** NotifyActivity 實例，每個實例各自呼叫 `InsertNotify()` 寫入通知，但 `InsertHistory()` 只在主活動上執行一次。

## 備註

- 工具箱顏色為黃色（#ffff00），與 WarningActivity 相同
- `SupportPrint` 屬性由 `CanPrint` 控制，預設為 `false`
- `AllUsers` 類型會查詢 `FlowHistoryHelper.SelectDetail()` 取得所有歷程使用者，再加上當前使用者，去重後逐一通知
- 與 WarningActivity（預警）的差異：NotifyActivity 僅發送待辦通知，不含預警主題/內容/級別等資訊
