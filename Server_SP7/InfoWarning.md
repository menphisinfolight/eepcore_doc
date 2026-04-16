# InfoWarning

> `EEPServerTools.Core/Components/InfoWarning.cs` — 365 行
> 繼承：`ServerComponent` → `Component`

## 用途

**警示通知元件**（Warning Notification）。

InfoWarning 用於發送系統警示通知給指定的使用者或角色。執行時透過關聯的 InfoCommand 查詢資料，對每筆查詢結果：
1. 將警示記錄寫入 `FlowWarning` 資料表。
2. 依據 SendType 設定，透過 Email、Line 或 Push 發送即時通知。

支援三種警示等級：C（concern 關注）、W（warning 警告）、S（serious 嚴重），預設為 W。

## JSON 設定範例

```json
{
  "type": "infowarning",
  "id": "warnOverdue",
  "subject": "逾期通知",
  "description": "訂單已逾期",
  "infocommand": "cmdOverdue",
  "formName": "OrderForm",
  "sender": "SYSTEM",
  "sendToType": "Role",
  "sendTo": "MGR,ADMIN",
  "sendToField": "",
  "sendType": "Mail",
  "level": "W",
  "keyFields": [
    { "title": "訂單編號", "field": "ORDER_NO" }
  ],
  "parameterFields": [
    { "title": "到期日", "field": "DUE_DATE" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **subject** | string | 文字 | — | 警示主旨 |
| **description** | string | 文字 | — | 警示描述文字 |
| **infocommand** | string | 元件選擇器 `[ControlEditor("infocommand")]` | — | 關聯的 InfoCommand 元件 ID |
| **keyFields** | List\<WarningField\> | 集合編輯器 `[CollectionEditor]` | [] | 識別鍵欄位，用於連結回原始表單資料 |
| **parameterFields** | List\<WarningField\> | 集合編輯器 `[CollectionEditor]` | [] | 參數欄位，附加於描述中顯示 |
| **formName** | string | 選單選擇器 `[MenuEditor("bootstrap")]` | — | 關聯的表單名稱（點擊警示可開啟此表單） |
| **sender** | string | 安全性選擇器 `[SecurityEditor("user")]` | — | 發送者使用者 ID |
| **sendToType** | SendToType | 列舉 | Role(0) | 發送對象類型：`Role`（角色）或 `User`（使用者） |
| **sendTo** | string | 動態安全性選擇器 `[DSecurityEditor("sendToType")]` | — | 發送對象 ID（多個以逗號分隔） |
| **sendToField** | string | 欄位選擇器 `[ColumnEditor]` | — | 從資料列取得發送對象的欄位名稱 |
| **sendType** | SendType | 列舉 | None(0) | 通知方式：`None`、`Mail`、`Line`、`Push` |
| **level** | string | 項目選擇器 `[ItemsEditor]` | "W" | 警示等級：C（關注）、W（警告）、S（嚴重） |

### WarningField 子類別屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **title** | string | 文字 | 欄位顯示標題 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | 資料欄位名稱 |

## 核心方法

### `Execute(string whereStr)` → `int`

主要執行方法，流程如下：

```
Execute(whereStr)
│
├─ 1. 建立 DatabaseHelper（含 Transaction）
├─ 2. 取得關聯的 InfoCommand，設定查詢條件
├─ 3. 執行 SQL 查詢取得資料
├─ 4. 逐筆處理查詢結果：
│     ├─ GetRoles(row) — 取得目標角色清單
│     ├─ GetUsers(row) — 取得目標使用者清單
│     ├─ InsertTable() — 寫入 FlowWarning 資料表
│     └─ SendNotify() — 發送即時通知
├─ 5. Commit Transaction
└─ 6. 回傳處理筆數
```

### `InsertTable(dbHelper, row, roles, users)`（私有）

將警示記錄寫入 `FlowWarning` 資料表，欄位包括：

| FlowWarning 欄位 | 值來源 |
|-------------------|--------|
| Type | 固定 "D" |
| Level | 元件 Level 屬性 |
| Subject | 元件 Subject 屬性 |
| Description | Description + ParameterFields 組合 |
| SenderID / SenderName | Sender 屬性，查 USERS 表取名稱 |
| Datetime | DateTime.Now |
| FormName | 元件 FormName 屬性 |
| KeyFields | 鍵欄位 JSON 字串 |
| PresentationFields | 鍵欄位的「標題:值」組合 |
| ProjectID | ClientInfo.Solution |
| IsFinish | false |
| RoleID / RoleName | 角色發送時填入 |
| UserID / UserName | 使用者發送時填入 |

每個角色和使用者各插入一筆記錄。

### `SendNotify(row, roles, users)`（私有）

根據 SendType 決定通知管道：
- **Mail**：從 USERS 表取得 Email，透過 `EmailHelper` 寄信（需在系統設定檔配置 email 區段）。
- **Line**：透過 `LineHelper` 發送 Line 通知（需在系統設定檔配置 Lines 區段）。
- **Push**：目前程式碼已註解，尚未啟用。

## 列舉定義

### SendToType

| 值 | 名稱 | 說明 |
|----|------|------|
| 0 | Role | 發送給角色（查 USERGROUPS 展開為使用者） |
| 1 | User | 直接發送給使用者 |

### SendType

| 值 | 名稱 | 說明 |
|----|------|------|
| 0 | None | 不發送通知（僅寫入 FlowWarning） |
| 1 | Mail | Email 通知 |
| 2 | Line | Line 通知 |
| 3 | Push | 推播通知（未實作） |

## 備註

- SendTo 和 SendToField 可同時使用，兩者的結果會合併。SendTo 為設計時指定的固定對象，SendToField 為執行時從資料列動態取得。
- 角色發送時，系統會查詢 USERGROUPS 表展開為個別使用者，再發送通知。
- FlowWarning 的 KeyFields 以 JSON 格式儲存，前端可據此開啟對應表單並定位到該筆資料。
- PresentationFields 為「標題:值」的逗號分隔格式，用於前端顯示摘要。
- Description 若有設定 ParameterFields，會在描述後附加括號包圍的參數值，格式如：`逾期通知(到期日:2026-04-01)`。
- GetUserName 和 GetRoleName 方法查詢 USERS / GROUPS 表時未帶條件篩選，可能為程式瑕疵（應以 ID 篩選）。
- Push 通知方式的實作已被註解，目前無法使用。
