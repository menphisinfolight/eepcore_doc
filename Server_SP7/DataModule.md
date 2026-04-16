# DataModule

> `EEPServerTools.Core/DataModule.cs` — 1507 行

## 用途

**資料模組核心**（Data Module Core）。

DataModule 是 EEP Core 伺服器端的核心類別，負責載入模組的 JSON 設定檔、管理所有元件（Components）、執行資料查詢與更新。每個模組（如 SystemTable、使用者自訂模組）都是一個 DataModule 實例。

實作 `IContainer` 介面，作為所有 Server 元件的容器。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `ClientInfo` | ClientInfo | 當前使用者的連線資訊（User、Database、Solution 等） |
| `Name` | string | 模組名稱（如 `"SystemTable"`、自訂模組名） |
| `Components` | List\<Component\> | 模組內所有元件（InfoCommand、UpdateComponent 等） |
| `DbHelper` | DatabaseHelper | 資料庫操作工具 |
| `Context` | HttpContext | HTTP 請求上下文 |
| `SYSTEM_TABLE` | const string | 常數 `"SystemTable"`，系統表模組名稱 |

## 建構與載入

### 建構函式

```csharp
// 標準建構：從 JSON 檔案載入元件
DataModule(ClientInfo clientInfo, string moduleName)

// 注入建構：直接傳入元件清單（不讀檔）
DataModule(ClientInfo clientInfo, string moduleName, IEnumerable<Component> components)
```

### JSON 檔案路徑規則

| 條件 | 路徑 |
|------|------|
| SystemTable + SQL Server | `design/server/SystemTable.json` |
| SystemTable + Oracle | `design/server/SystemTable.oracle.json` |
| SystemTable + MySQL | `design/server/SystemTable.mysql.json` |
| SystemTable + DB2 | `design/server/SystemTable.db2.json` |
| SystemTable + Informix | `design/server/SystemTable.informix.json` |
| 自訂模組 | `design/server/{Solution}/{ModuleName}.json` |

### 元件建立流程

```
LoadComponents(dbType)
  → 讀取 JSON 檔案 → JArray
  → 遍歷每個 JObject → CreateComponent(item)
    → 以 item["type"] 反射找到對應類別（如 "infocommand" → InfoCommand）
    → Activator.CreateInstance() 建立實例
    → comp.LoadProperties(item) 載入 JSON 屬性
    → comp.Container = this
```

## 核心方法

### 資料查詢

```csharp
DataTable GetDataset(string commandName, QueryOptions options)
```

| 步驟 | 說明 |
|------|------|
| 1 | 取得 InfoCommand 元件 |
| 2 | 若有 `ParentTable` → 透過 InfoDataSource 取得主從關聯 SQL |
| 3 | 若無 → 由 InfoCommand.GetSql() 產生 SQL |
| 4 | CacheSynchronize 處理：`Smart` 模式只查新資料、`None` 回傳 null |
| 5 | `dbHelper.ExecuteDataTable()` 執行查詢 |
| 6 | 觸發 `OnAfterExecuteSQL` 事件 |

### 資料更新

```csharp
object UpdateDataset(string commandName, JArray datas, UpdateOptions options)
```

| 步驟 | 說明 |
|------|------|
| 1 | 遍歷 datas 中每個 JObject（table + inserted/updated/deleted） |
| 2 | 找到對應的 UpdateComponent |
| 3 | 若為明細（IsDetail）→ 透過 InfoDataSource 取得父值 |
| 4 | `updateComponent.GetSqls(rows)` 產生 INSERT/UPDATE/DELETE SQL |
| 5 | `dbHelper.ExecuteNonQuery(sqls)` 批次執行 |
| 6 | 處理 Identity 欄位（取回自動遞增 ID） |
| 7 | Transaction Commit 或 Rollback |
| 8 | 觸發 `OnAfterApplied` 事件 |

### 自訂方法呼叫

```csharp
object CallMethod(string methodName, object parameters)
object CallProcessorMethod(string id, JObject parameters)
```

| 方法 | 說明 |
|------|------|
| `CallMethod` | 載入模組 DLL（`design/server/{Solution}/{ModuleName}.Core.dll`），反射呼叫指定方法 |
| `CallProcessorMethod` | 呼叫 Processor 介面卡，傳入連線資訊和參數 |
| `CheckMethod` | 檢查是否允許非登入狀態呼叫（ServiceManager.NonlogonMethods） |

### Excel 匯出/匯入

```csharp
void ExportXlsx(string filePath, string commandName, JArray columns, string title, JArray relations, QueryOptions options)
object ImportXlsx(string filePath, string commandName, ImportOptions options)
```

| 功能 | 說明 |
|------|------|
| **ExportXlsx** | 查詢資料 → 處理關聯欄位對應 → NPOI 產生 xlsx（支援數值/日期格式化） |
| **ImportXlsx** | 讀取 xlsx → 欄位名對應（標題 ↔ schema）→ 產生 INSERT/UPDATE SQL → 批次執行 |

### 預設值

```csharp
string GetDefaultValue(string defaultValue)
```

支援三種預設值類型：

| 類型 | 格式 | 說明 |
|------|------|------|
| `constant` | `constant["值"]` | 固定值 |
| `varaible` | `varaible["today"]` | 變數：today、now、firstday、lastday、User、Solution 等 |
| `function` | `function["方法名"]` | 呼叫自訂方法取得值 |

varaible 支援的日期變數：

| 變數 | 說明 |
|------|------|
| `today` | 今天（yyyy/MM/dd） |
| `now` | 現在（yyyy/MM/dd HH:mm:ss） |
| `todayc8` | 今天（yyyyMMdd） |
| `firstday` / `lastday` | 本月第一天/最後一天 |
| `firstdaylm` / `lastdaylm` | 上月第一天/最後一天 |
| `firstdayty` / `lastdayty` | 今年第一天/最後一天 |
| `firstdayly` / `lastdayly` | 去年第一天/最後一天 |

### 流程提交

```csharp
object SubmitFlow(JToken row, JToken parameters)
```

將表單資料提交給流程引擎（Start/Submit），透過 FlowHelper.CallFlowMethod 執行。

### 其他

| 方法 | 說明 |
|------|------|
| `GetComponent<T>(id)` | 取得指定 ID 和類型的元件 |
| `GetComponents<T>()` | 取得指定類型的所有元件 |
| `GetDetailCommands(commandName)` | 取得主表的所有明細 InfoCommand（透過 InfoDataSource 關聯） |
| `CreateDatabaseHelper()` | 建立資料庫連線 |
| `Echo(text)` | 偵錯日誌（寫入 DebuggerLog，記錄呼叫方法名） |
| `GetConfig()` | 取得系統設定（ConfigHelper.GetParam） |

## 元件存取模式

```
外部呼叫 → DataModule
         ├── GetDataset("commandName", options)
         │   → InfoCommand.GetSql() → 查詢
         │   → InfoDataSource.GetDetailSql() → 主從查詢
         │
         ├── UpdateDataset("commandName", datas, options)
         │   → UpdateComponent.GetSqls(rows) → 更新
         │   → InfoDataSource.GetParentValues() → 父值
         │
         └── CallMethod("methodName", params)
             → 載入 DLL → 反射呼叫
```

## 備註

- SAP 相關功能（SapContent）已被註解掉，但程式碼保留在檔案中。
- 模組 DLL 每次呼叫都重新載入（`File.ReadAllBytes` + `Assembly.Load`），舊版的快取機制也被註解。
- `InvokeMethod` 會建立一個新的 DataModule 實例（透過反射呼叫 DLL 中的建構函式），但保留原始的 Components 參照。
- Excel 匯入支援 `UpdateIfExists` 模式：有主鍵資料時 UPDATE，無則 INSERT。
- Informix 有特殊的參數化處理（MarkParameterValue）。
