# FlowHelper

> `EEPServerTools.Core/Utility/FlowHelper.cs` — 851 行

## 用途

**流程引擎操作工具**（Flow Engine Helper）。

FlowHelper 是 EEP Core 伺服器端的流程操作輔助類別，負責封裝與流程引擎（Flow）的所有互動。包括：流程的啟動、送出、退回、抽回、駁回、加簽等操作，以及流程查詢、延遲處理、通知發送（Email / Push / LINE）等功能。

透過 `Flow` Adapter 與底層流程引擎通訊，並根據操作結果組裝回傳訊息與發送通知。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Context` | HttpContext | HTTP 請求上下文 |
| `ClientInfo` | ClientInfo | 當前使用者的連線資訊 |
| `FileDir` | string | 檔案目錄路徑 |

## 核心方法

| 方法 | 說明 |
|------|------|
| `CallFlowMethod(IFormCollection)` | 流程操作主入口，根據 method 參數分派不同流程動作 |
| `CallDelayMethod(dynamic)` | 延遲批次處理（延遲審核、延遲寄信），回傳處理統計 |
| `QueryFlow(IFormCollection)` | 查詢流程資料（支援分頁），type 決定查詢類型 |
| `QueryData(IFormCollection)` | 查詢流程相關資料（待辦、代理等），支援分頁與代理人設定 |
| `SendNotify(JObject, JObject, JObject, string)` | 通知發送主入口，處理 ToDo / Notify / Warning 三類通知 |

## CallFlowMethod 流程分派邏輯

`CallFlowMethod` 從 `IFormCollection` 中擷取流程參數（method、FlowID、InstanceID、ActivityID 等），合併使用者連線資訊後呼叫 `Flow.CallFlowMethod`。根據 `method` 值走不同的後處理邏輯：

### 直接回傳結果（不處理通知）

| method | 說明 |
|--------|------|
| `Prepare` | 準備流程（取得流程定義） |
| `DeleteNotify` | 刪除通知 |
| `Preview` | 預覽流程 |
| `Retake` | 抽回（撤回已送出的表單） |
| `ReadComment` | 讀取簽核意見 |
| `ConfirmWarning` | 確認預警 |
| `AddPhrase` | 新增常用語 |
| `DeletePhrase` | 刪除常用語 |

### 處理通知並回傳訊息

| method | 說明 |
|--------|------|
| `Start` | 啟動流程，回傳 `{ InstanceID, message }` |
| `Submit` | 送出（審核通過） |
| `PlusApprove` | 加簽核准 |
| `Return` | 退回上一關 |
| `Reject` | 駁回（終止流程） |

這些操作會：
1. 解析回傳的 Instance、Next（下一關）、Waiting（等待中）
2. 呼叫 `SendNotify` 發送通知
3. 若 Next 和 Waiting 皆為空 → 流程結束（依 Status=7 判斷為駁回或正常結束）
4. 否則組裝「已送至 XX 關卡 / 等待 XX 關卡」訊息

### 預設分支（含 Notify）

其他未列舉的 method（如 `Notify`）：處理通知後，將 `r["Notify"]` 指派為 `r["Next"]`，回傳關卡訊息。

## 錯誤處理

| 錯誤訊息 | 處理方式 |
|----------|----------|
| `userNotFound:{user}` | 轉為多語系的「使用者不存在」EEPException |
| `roleNotFound:{role}` | 轉為多語系的「角色不存在」EEPException |
| `flowExist` | 轉為多語系的「流程已存在」EEPException |
| 其他 | 原樣拋出 |

## 通知發送流程

```
SendNotify(item, result, parameter, url)
  ├── result["Next"]  → 待辦通知（ToDo）
  │   └── 逐筆 activity → SendActvityNotify(instance, config, item, activity, "ToDo")
  ├── result["Notify"] → 知會通知（Notify）
  │   └── 逐筆 activity → SendActvityNotify(instance, config, item, activity, "Notify")
  └── result["Warning"] → 預警通知
      └── 逐筆 activity → SendActvityWarning(instance, config, item, activity)
```

### 通知管道

每個 Instance 可獨立設定三種通知管道（由 Instance 屬性控制）：

| 管道 | 判斷屬性 | 實作方法 | 說明 |
|------|----------|----------|------|
| Push | `SendPush` | `SendPush()` | 呼叫 `sendPushNotification` 自訂方法推播 |
| Email | `SendNotify` | `SendMail()` | 透過 `EmailHelper` 寄送 HTML 郵件 |
| LINE | `SendLine` | `SendLine()` | 透過 `LineHelper` 發送 LINE 訊息 |

通知內容透過 `ConfigHelper.GetFlowConfig()` 取得 `emailSubject` / `emailBody` 範本，以 `<%=...%>` 格式進行欄位替換。

### 訊息範本格式

```
<%=_type%>          → 通知類型（待辦/知會/預警）
<%=_instance.FlowText%> → 流程名稱
<%=_activity.Text%>     → 關卡名稱
<%=_history.toTable('ActivityText','UserName','Status','Remark','Datetime')%>
                        → 簽核歷史 HTML 表格
<%=new Date()%>         → 當前日期時間
```

## 輔助方法

| 方法 | 說明 |
|------|------|
| `SetTableName(JObject)` | 從 Parameter 中的 PROVIDER_NAME 反查實際 TABLE_NAME |
| `GetMessage(JObject)` | 組裝「送至 XX / 等待 XX」的回傳訊息，含關卡名稱與使用者資訊 |
| `GetSendText(JObject, JArray)` | 格式化送出對象文字（角色名或使用者名 + GroupUsers 資訊） |
| `GetClientParam()` | 組裝流程引擎所需的連線參數（user、database、conn_str 等） |
| `GetHistory(string)` | 取得流程簽核歷史（依 InstanceID），結果快取於 `HistoryTabls` |
| `FormatText(string, JObject)` | 以 Regex 替換 `<%=...%>` 範本標記 |
| `FormatValue(JObject, string, Dictionary)` | 格式化狀態碼為多語系文字（0=送出、1=核准、2=退回 等） |
| `ToHistoryTable(JArray, IEnumerable)` | 將簽核歷史轉為 HTML `<tr>` 表格列 |

## 狀態碼對應

| 狀態碼 | 意義 |
|--------|------|
| 0 | 送出（submit） |
| 1 | 核准（approve） |
| 2 | 退回（back） |
| 3 | 抽回（retake） |
| 5 | 加簽（plus） |
| 7 | 駁回（reject） |
| 8 | 啟動流程（startFlow） |
| 9 | 結案（finished） |

## 附加：IFormCollectionExtensions

檔案末尾包含 `IFormCollectionExtensions` 內部靜態類別，提供 `GetValue<T>` 擴充方法，從 `IFormCollection` 安全取值並轉型，支援 bool 特殊處理。

## 備註

- 通知發送時每筆使用者之間會 `Thread.Sleep(100)`，避免過快觸發限流。
- SSO 連結透過 `ConfigHelper.GetSSOKey` 產生，URL 格式為 `{host}/flow/mail?p={encoded_sso_key}`。
- `CallDelayMethod` 用於排程批次處理，支援 `SendMail`（延遲寄信）和延遲審核兩種模式。
- `GetHistory` 使用 `HistoryTabls` 字典做 Instance 層級快取，避免同一次通知發送中重複查詢。
- Push 通知的 Parameter 中單引號會被替換為 `\^\^\^`（`row["Parameter"]`），可能為前端解析需求。
- `GetClientParam` 中 `user_str`（使用者專屬連線字串）已被註解。
