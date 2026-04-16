# ApproveActivity（逐級簽核活動）

## 檔案資訊

| 項目 | 路徑 | 行數 |
|------|------|------|
| 設計端 | `EEPFlowTools/Activities/ApproveActivity.cs` | 61 行 |
| 引擎端 | `EEP.Flow/Activities/ApproveActivity.cs` | 204 行 |

繼承鏈：
- **ApproveActivity** 繼承 `SequenceActivity` → `CompositedActivity` → `Activity`
- **ApproveBranchActivity** 繼承 `StandActivity` → `Activity`

---

## 用途

ApproveActivity 是 EEP Flow 流程引擎中的**逐級簽核活動元件**，用於實現「依組織層級逐級往上簽核」的場景。

與 StandActivity 的單一關卡不同，ApproveActivity 是一個**複合型容器活動**，內含多個 `ApproveBranchActivity` 子關卡，每個子關卡代表一個組織層級的簽核。引擎會依序（Sequence）執行這些子關卡，從低階主管往高階主管逐級簽核。

核心特性：
- **動態層級展開**：根據 ApproveRights 設定，依組織層級自動展開簽核關卡
- **條件過濾**：每個層級可設定 Expression（條件運算式）或 Function（伺服器方法）來決定是否需要該層級簽核
- **順序執行**：子關卡按順序執行（繼承 SequenceActivity），前一層核准後才進入下一層
- **自動跳過**：若某層級找不到對應的主管角色，自動跳過該層級

---

## JSON 設定範例

```json
{
  "type": "ApproveActivity",
  "ID": "approve1",
  "Text": "主管逐級核准",
  "SendTo": "ApplicantManager",
  "Parameters": "",
  "PlusMode": "Disabled",
  "ExpDay": 5,
  "CanEdit": true,
  "CanReturn": true,
  "CanAgent": true,
  "CanReject": false,
  "ApproveRights": [
    {
      "Level": "L3",
      "Expression": "",
      "Function": ""
    },
    {
      "Level": "L2",
      "Expression": "Amount > 50000",
      "Function": ""
    },
    {
      "Level": "L1",
      "Expression": "",
      "Function": "CheckNeedCEO"
    }
  ]
}
```

上例表示：
- L3 主管：必定簽核
- L2 主管：僅在金額超過 50,000 時需要簽核
- L1 主管：由伺服器方法 `CheckNeedCEO` 決定是否需要簽核

---

## 設計介面屬性

以下屬性來自 `EEPFlowTools/Activities/ApproveActivity.cs`：

| 屬性 | 型別 | 預設值 | 設計器 Attribute | 說明 |
|------|------|--------|------------------|------|
| ApproveRights | `List<ApproveRight>` | 空集合 | `[CollectionEditor("netflow", "approveActivity", "approveRight", "level")]` | 簽核權限集合 |
| SendTo | `string` | — | `[ValueOptionEditor("netflow", "approveActivity", true)]` | 起始角色來源 |
| Parameters | `string` | — | （無） | 自訂參數 |
| PlusMode | `PlusMode` | `Disabled` | （無） | 加簽模式 |
| ExpDay | `double` | `0` | `[NumberboxEditor(0, true, 0, 2)]` | 逾時天數 |
| CanEdit | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許編輯 |
| CanReturn | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許退回 |
| CanAgent | `bool` | `true` | `[CheckboxEditor(true)]` | 是否允許代理 |
| CanReject | `bool` | `false` | （無） | 是否允許作廢 |

### ApproveRight 子類別屬性

| 屬性 | 型別 | 設計器 Attribute | 說明 |
|------|------|------------------|------|
| Expression | `string` | （無） | 條件運算式（由 Instance.JudgeExpression 評估） |
| Function | `string` | `[ServerMethodEditor]` | 伺服器方法名稱（回傳 true/false） |
| Level | `string` | `[SecurityEditor("orgLevel", "LEVEL_NO", "LEVEL_DESC")]` | 組織層級編號 |

### SendTo 內部 IValueType 子類別

| 類別 | Field 子屬性 | 說明 |
|------|-------------|------|
| `Manager` | — | 目前使用者的主管 |
| `ApplicantManager` | — | 申請人的主管 |
| `RefManager` | `Field` | 從表單欄位取得角色，再往上找主管 |

> 注意：ApproveActivity 的 SendTo 只支援 Manager 系列（需要往上找主管），不支援 StandActivity 中的 Applicant、RefRole、RefUser。

---

## 引擎執行邏輯

### ApproveActivity 類別（第 14-52 行）

ApproveActivity 本身是一個輕量容器，核心邏輯在 `Children` 屬性的動態計算：

#### Children — 動態過濾子活動（第 31-51 行）

```csharp
public override List<Activity> Children
{
    get
    {
        if (approveActivities == null)
        {
            approveActivities = new List<Activity>();
            ChildActivities.ForEach(c => c.Instance = Instance);
            var childs = ChildActivities.OfType<ApproveBranchActivity>()
                .Where(c => c.Invoke());  // 關鍵：條件過濾
            foreach (var child in childs)
            {
                child.Parameters = Parameters;
                approveActivities.Add(child);
            }
        }
        return approveActivities;
    }
}
```

邏輯：
1. 將所有 ChildActivities 設定 Instance
2. 篩選型別為 `ApproveBranchActivity` 的子活動
3. **呼叫 `Invoke()` 方法進行條件過濾**，只保留結果為 true 的子活動
4. 設定每個子活動的 Parameters
5. 快取結果（只計算一次）

#### Start() — 繼承自 SequenceActivity

ApproveActivity 繼承 `SequenceActivity.Start()`，邏輯為：
- 若自身已完成，跳至下一個活動
- 否則，啟動 `Children` 集合中的第一個子活動

### ApproveBranchActivity 類別（第 55-204 行）

ApproveBranchActivity 繼承自 `StandActivity`，代表逐級簽核中的**每一層級**。

#### Invoke() — 條件判斷（第 79-90 行）

決定此層級是否需要簽核：

1. 若有 `Expression`：呼叫 `Instance.JudgeExpression(Expression)` 評估條件
2. 若有 `Function`：呼叫 `Instance.Invoke(Function)` 執行伺服器方法，比對回傳值是否為 `True`
3. 若兩者皆無：預設為 `true`（需要簽核）

#### Start() — 啟動簽核層級（第 167-191 行）

覆寫 StandActivity 的 Start()，邏輯如下：

1. **取得起始角色**（`StartRole` 屬性）：
   - 若為第一個子活動（index = 0）：根據父活動 ApproveActivity 的 SendTo 決定
     - `Manager`：取目前使用者的角色
     - `ApplicantManager`：取申請人的角色
     - `RefManager`：從表單欄位取角色
   - 若非第一個：取**前一個子活動**的 `CurrentRole`（即前一層的角色）
2. **解析主管角色**（非退回/收回時）：
   - 呼叫 `OrgInfo.GetManagerID(currentRole, Level, OrgKind)` 取得該層級的主管角色
3. **啟動或跳過**：
   - 若找到主管角色（Role 非空）：清空 SendTo，呼叫 `base.Start()` 建立待辦
   - 若找不到（Role 為空）：直接標記為 Executed，記錄 `_currentRole`，跳至下一個子活動

#### ActivityText — 活動文字（第 95-102 行）

覆寫為 `{Text}_{組織層級名稱}`，例如「主管核准_部門主管」。

#### IsWaiting — 是否需要等待（第 196-202 行）

覆寫為只檢查 `Role` 是否非空（忽略 User），因為逐級簽核只使用角色。

#### StartRole — 起始角色解析（第 104-149 行，私有屬性）

逐級簽核的核心邏輯之一，決定「從哪個角色開始往上找主管」：

| 情境 | 角色來源 |
|------|----------|
| 第一層 + SendTo = Manager | 目前使用者的角色 |
| 第一層 + SendTo = ApplicantManager | 流程申請人的角色 |
| 第一層 + SendTo = RefManager | 表單欄位指定的角色 |
| 第一層 + SendTo 為空/其他 | 目前使用者的角色（預設） |
| 非第一層 + 前一層有 CurrentRole | 前一層的 CurrentRole |
| 非第一層 + 前一層無 CurrentRole | 目前使用者的角色（預設） |

---

## 與官方文件差異

### 官方列出 10 項屬性

官方文件：Title, ApproveRights (collection), SendTo, Parameters, PlusMode, ExpDay, CanEdit, CanReturn, CanAgent, CanReject

### 原始碼有但官方未列出的屬性

ApproveActivity 本身無額外屬性差異。但 ApproveBranchActivity 繼承自 StandActivity，因此每個子關卡擁有 StandActivity 的所有引擎屬性（ReturnTo、Duration、OnSubmit、SubmitResult 等），這些在官方文件中未提及。

### 官方有但原始碼對應說明

- **Title**：與 StandActivity 相同，對應基底類別的 `Text` 屬性。
- **ApproveRights**：官方以 collection 描述，原始碼中為 `List<ApproveRight>`，每個 ApproveRight 包含 Expression、Function、Level 三個屬性。

### CanPrint 差異

StandActivity 有 `CanPrint` 屬性，但 ApproveActivity 的設計端**沒有** CanPrint。這表示逐級簽核的子關卡**不支援個別控制列印權限**。官方文件也未列出 CanPrint，與原始碼一致。

---

## 備註

1. **ApproveActivity vs StandActivity 的選擇**：
   - 若只需一個固定的簽核關卡，使用 StandActivity
   - 若需要根據組織層級逐級往上簽核（例如：組長 → 課長 → 經理 → 副總），使用 ApproveActivity

2. **ApproveBranchActivity 繼承 StandActivity**：每個簽核層級本質上就是一個 StandActivity，因此支援加簽、轉簽、退回、代理等所有 StandActivity 的功能。

3. **條件過濾的時機**：ApproveRights 的條件在 `Children` 屬性首次存取時評估（lazy + cached），之後不會重新評估。這表示條件是在活動啟動時一次性決定的。

4. **自動跳過機制**：若 `OrgInfo.GetManagerID()` 找不到某層級的主管（回傳空值），該層級會自動跳過（標記 Executed，前進到下一層）。這在組織層級不完整時提供了容錯能力。

5. **SendTo 只支援 Manager 系列**：與 StandActivity 不同，ApproveActivity 的 SendTo 僅支援 Manager、ApplicantManager、RefManager 三種類型，因為逐級簽核必須基於角色/組織結構往上找主管。

6. **CurrentRole 的傳遞**：當某層級被跳過時，`_currentRole` 會保留跳過前的角色值，確保下一層級能正確地從該角色繼續往上找主管。這是逐級簽核連續性的關鍵機制。

7. **Parameters 繼承**：ApproveActivity 的 Parameters 會傳遞給每個 ApproveBranchActivity 子活動，確保所有層級共用相同的參數設定。
