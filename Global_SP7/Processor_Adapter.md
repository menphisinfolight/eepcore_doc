# Processor (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/Processor.cs` |
| 行數 | 702 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

Processor（自動化處理器）執行引擎的 Adapter 層，負責：
1. 呼叫 Processor 的 Start / Debug 方法
2. 將流程設計 JSON 轉為 XML（SaveToXml）
3. WebService 活動預覽
4. 提供 `ProcessorProvider`（IClientInfoProvider + IMethodProvider）實作

## 主要類別

### Processor

| 方法 | 說明 |
|------|------|
| `CallProcessorMethod(param)` | 執行指定 Processor |
| `CallDebugMethod(param)` | 偵錯模式（debugStart / debugStep / debugTo / debugStop / executeExpression） |
| `SaveToXml(param)` | 將設計器 JSON 轉為 Processor XML 定義 |
| `PreviewService(param)` | 預覽 WebServiceActivity 結果 |
| `submitFlow(row, parameters)` | 從 Processor 內部發起簽核流程 |

### ConnStrProvider

`IConnStrProvider` 實作，根據 database 名稱取得連線字串。

### ProcessorProvider

實作 `IClientInfoProvider` + `IMethodProvider`：

| 方法 | 說明 |
|------|------|
| `GetUserName / GetGroupName` | 查 USERS / GROUPS 表 |
| `GetUsers(group)` | 取得群組成員 |
| `Invoke(instance, methodName)` | 透過 DataModule 反射呼叫方法 |
| `InvokeComponent(...)` | 透過 DataModule 呼叫指定元件方法 |
| `GetEmails(users)` | 取得使用者 Email（支援直接 email 或查 USERS） |
| `SendMail(...)` | 發送 SMTP 郵件 |

## 備註

- `SaveToXml` 中的活動型別透過反射從 `EEP.Processor.Activities` 命名空間取得
- 支援複合活動（CompositedActivity），遞迴處理子活動
- 資料庫目前僅支援 SQL Server（`ProcessorSqlDatabase`）
