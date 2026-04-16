# Flow (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/Flow.cs` |
| 行數 | 1040 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

簽核流程（NetFlow）的 Adapter 層，負責流程方法呼叫、流程查詢、流程資料查詢、設計 JSON 轉 XML，以及 `FlowProvider`（實作多個流程介面）。

## 主要類別

### Flow

| 方法 | 說明 |
|------|------|
| `CallFlowMethod(param)` | 流程操作分派（Prepare / Start / Submit / Return / Retake / Notify / Plus / PlusAgent / PlusApprove / PlusReturn / PlusTransfer / Reject / Delete / Preview / Comment / ReadComment） |
| `QueryFlow(param)` | 流程查詢分派（ToDo / History / Notify / End / Comment / AllToDo / AllEnd / CurrentToDo / CurrentNotify / Detail / CurrentComment / Defination / PrevivousActivities） |
| `QueryData(param)` | 流程資料查詢（Groups / Users / UserGroups / GroupUsers / UserInfos） |
| `SaveToXml(param)` | 將設計器 JSON 轉為流程 XML，支援 ApproveActivity 的 approveBranch 子活動 |

### FlowProvider

實作 `IClientInfoProvider` / `IAgentInfoProvider` / `IOrgInfoProvider` / `IEventProvider` / `IMethodProvider`：

| 功能群組 | 方法 |
|----------|------|
| 使用者/群組 | `GetUserName` / `GetGroupName` / `GetUsers` / `GetUserGroups` / `GetGroupUsers` |
| 代理人 | `IsUserAgented` / `FilterAgentedGroups` / `GetAgentGroups` / `GetAgentUsers` |
| 組織 | `GetManagerID`（遞迴向上查主管）/ `GetOrgLevelName` |
| 事件 | `OnCreate` / `OnPrepare` / `OnStart` / `OnSubmit` / `OnReturn` / `OnReject` / `OnDelete` / `OnEnd` |
| 方法呼叫 | `Invoke`（透過 DataModule 呼叫外部方法） |

### FlowFlag 機制

透過 `CheckFlowFlag` / `UpdateFlowFlag` 維護表單的流程狀態：

| FlowFlag 值 | 狀態 |
|-------------|------|
| P:InstanceID | 準備中 |
| N:InstanceID | 進行中 |
| X:InstanceID | 已退回 |
| Z:InstanceID | 已結案（可重啟） |
| 空 | 已刪除 |

### QueryParam

解析查詢參數（FlowText / Parameter / DatetimeFrom / DatetimeTo / Sender / User / Role / IsRead）為 SQL 條件。

## 備註

- 支援三種資料庫：SQL Server（FlowSqlDatabase）、MySQL（FlowMySqlDatabase）、Oracle（FlowOracleDatabase）
- Oracle 參數處理為字串替換（非 parameterized query），使用正則替換 `:paramName`
- 流程事件（OnStart / OnEnd 等）會自動更新 FlowFlag 欄位
