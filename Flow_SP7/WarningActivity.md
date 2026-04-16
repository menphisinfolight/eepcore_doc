# WarningActivity（預警活動 / Prewarning）

> 設計端：`EEPFlowTools/Activities/WarningActivity.cs`（60 行）— 類別名稱 `PrewarningActivity`
> 引擎端：`EEP.Flow/Activities/WarningActivity.cs`（218 行）— 類別名稱 `PrewarningActivity`

## 用途

預警活動，用於在流程進行中向指定角色或使用者發送**預警訊息**。與 NotifyActivity 類似，不需要收件者簽核，但額外攜帶預警主題（Subject）、描述（Description）和級別（Level）等資訊。工具箱中顯示名稱為 **Prewarning**。繼承自 `ContinuedActivity`，執行後自動銜接後續活動。

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 預設值 | 說明 |
|------|------|--------|--------|------|
| Title | string | （繼承自 Activity） | — | 活動標題（顯示名稱） |
| Role | string | `FlowRoleEditor` | — | 群組編號（接收預警的角色） |
| User | string | `FlowUserEditor` | — | 使用者編號（接收預警的使用者） |
| SendTo | string | `ValueOptionEditor("netflow", "prewarningActivity", true)` | — | 接收者類型（下拉選單） |
| AllowEmpty | bool | — | `false` | 允許接收者為空白 |
| Subject | string | — | — | 預警主題 |
| Description | string | — | — | 預警描述內容 |
| Level | string | `ItemsEditor(["C","W","S"], ["concern","warning","serious"], "W")` | `"W"` | 預警級別 |

### Level 預警級別

| 值 | 標籤 | 說明 |
|----|------|------|
| `C` | concern | 關注 |
| `W` | warning | 警告（預設） |
| `S` | serious | 嚴重 |

### SendTo 類型（IValueType 內部類別）

| 類型 | 說明 | 額外屬性 |
|------|------|----------|
| `Applicant` | 申請人（流程啟動者） | 無 |
| `Manager` | 當前簽核者的主管 | 無 |
| `ApplicantManager` | 申請人的主管 | 無 |
| `RefRole` | 參考欄位指定的角色 | `Field`：欄位名稱 |
| `RefUser` | 參考欄位指定的使用者 | `Field`：欄位名稱 |
| `RefManager` | 參考欄位指定角色的主管 | `Field`：欄位名稱 |

> 注意：與 NotifyActivity 不同，WarningActivity 有 `RefManager` 但沒有 `AllUsers`。

## 引擎執行邏輯

### Invoke() 主流程

```
1. 若 SendTo 有值，解析類型（正則：(\w+)(\['(\w+)'\])?）
   ├─ Applicant        → User = 流程啟動者
   ├─ Manager          → Role = 當前簽核者的上級主管
   ├─ ApplicantManager → Role = 申請人的上級主管
   ├─ RefRole          → 從欄位取得多個角色 → 各自建立 PrewarningActivity → InsertWarning → 回傳
   ├─ RefUser          → 從欄位取得多個使用者 → 各自建立 PrewarningActivity → InsertWarning → 回傳
   └─ 其他             → 拋出 NotSupportedException

2. 若 SendTo 為空、且 Role 與 User 皆為空：
   ├─ AllowEmpty = true  → 回傳空清單（跳過預警）
   └─ AllowEmpty = false → 拋出 NotSupportedException

3. 一般情況：建立單一 PrewarningActivity → InsertWarning() → 回傳
```

### 引擎端額外屬性（唯讀計算）

| 屬性 | 說明 |
|------|------|
| `FormName` | 從 Instance.Parameter JSON 取得 `WEBFORM_NAME` |
| `KeyFields` | 從 Instance.Parameter JSON 取得 `FORM_KEYS`，組合成鍵值 JSON |
| `PresentationFields` | 從 Instance.Parameter JSON 取得 `FORM_PRESENTATION_CT`（展示用欄位） |

### 關鍵方法

| 方法 | 說明 |
|------|------|
| `InsertWarning()` | 透過 `FlowWarningHelper.Insert()` 寫入預警記錄 |
| `CreateWarningActivity()` | 以 `MemberwiseClone` 複製自身，替換 ID / User / Role，用於多人預警場景 |
| `GetDescription()` | 組合 Description + PresentationFields 為完整描述文字 |
| `ToObject()` | 輸出為 `object[]`：`{ ID, ActivityText, Role, GroupName, User, UserName, 0, false, Subject, Description }` |

### 與 NotifyActivity 的差異

| 項目 | NotifyActivity | WarningActivity |
|------|---------------|-----------------|
| 寫入目標 | `FlowNotifyHelper`（通知） | `FlowWarningHelper`（預警） |
| 額外資訊 | 無 | Subject / Description / Level |
| SendTo 類型 | 含 AllUsers | 含 RefManager，無 AllUsers |
| LogHistory | 有（可選） | 無 |
| CanPrint | 有（可選） | 無 |
| ToObject 輸出 | 8 個欄位 | 10 個欄位（多 Subject, Description） |

## 備註

- 工具箱顏色為黃色（#ffff00），工具箱顯示名稱為 **Prewarning**
- 引擎與設計端的類別名稱皆為 `PrewarningActivity`，但檔案名稱為 `WarningActivity.cs`
- `RefManager` 類型在設計端有定義（含 Field 屬性），但引擎端 `Invoke()` 的 switch 中**未處理此類型**，執行時會進入 default 分支拋出 `NotSupportedException`（可能為尚未實作或由其他機制處理）
- 預警描述 `GetDescription()` 會自動附加 `PresentationFields`，格式為 `Description(PresentationFields)`
