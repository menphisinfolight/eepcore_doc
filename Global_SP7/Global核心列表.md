# EEP Core Global 核心層（SP7）

> 原始碼位置：`EEPGlobal.Core/`
> 角色：Controller → **Provider（本層）** → DataModule → 資料庫

EEPGlobal.Core 是 EEP Core 的業務邏輯中間層，負責接收 Controller 的請求、進行業務處理、再呼叫 DataModule 執行資料操作。

## Provider — 業務邏輯提供者

### 核心 Provider

| Provider | 行數 | 說明 |
|----------|------|------|
| [BaseProvider](./BaseProvider.md) | 832 | Provider 基底類別（工廠模式，依 type 分派） |
| [AccountProvider](./AccountProvider.md) | 1441 | 帳號管理（登入驗證、密碼、SSO、LINE、Azure AD） |
| [DataProvider](./DataProvider.md) | 1040 | 資料操作（查詢/更新/匯出匯入/自訂報表/記錄鎖定） |
| [SecurityProvider](./SecurityProvider.md) | 677 | 安全管理（使用者/群組權限匯出、選單權限） |
| [MenuProvider](./MenuProvider.md) | 2225 | 選單管理（選單 CRUD、打包、簽入簽出、版本控制） |

### 檔案與報表

| Provider | 行數 | 說明 |
|----------|------|------|
| [FileProvider](./FileProvider.md) | 1641 | 檔案處理（上傳/下載/CSS/GeoJSON/SVG） |
| [WordProvider](./WordProvider.md) | 697 | Word 報表產生 |
| [ExcelProvider](./ExcelProvider.md) | 846 | Excel 報表產生/匯入匯出 |
| [FileListProvider](./FileListProvider.md) | 205 | 報表檔案清單 |
| [FontProvider](./FontProvider.md) | 85 | 字型管理 |

### 設計工具

| Provider | 行數 | 說明 |
|----------|------|------|
| [DatabaseProvider](./DatabaseProvider.md) | 547 | 資料庫管理（連線設定、資料表結構） |
| [SchemaProvider](./SchemaProvider.md) | 532 | Schema 管理（資料表欄位結構） |
| [TableProvider](./TableProvider.md) | 400 | 資料表設計 |
| [ControlProvider](./ControlProvider.md) | 277 | 控制項設計 |
| [ColdefProvider](./ColdefProvider.md) | 147 | COLDEF 欄位定義管理 |
| [VersionControlProvider](./VersionControlProvider.md) | 249 | 版本控制（簽入/簽出） |
| [UserTableProvider](./UserTableProvider.md) | 294 | 使用者資料表設計 |
| [UserMenuProvider](./UserMenuProvider.md) | 85 | 使用者選單設計 |
| [MenuGroupProvider](./MenuGroupProvider.md) | 82 | 選單群組管理 |

### 流程

| Provider | 行數 | 說明 |
|----------|------|------|
| [FlowProvider](./FlowProvider.md) | 254 | 流程操作（待辦/歷史/通知查詢） |
| [NetFlowProvider](./NetFlowProvider.md) | 79 | EEP.NET Flow 操作 |
| [FlowAnalysisProvider](./FlowAnalysisProvider.md) | 212 | 流程分析統計 |

### AI / 訊息 / 其他

| Provider | 行數 | 說明 |
|----------|------|------|
| [ChatProvider](./ChatProvider.md) | 1814 | AI 聊天（Azure OpenAI / Claude / OpenAI） |
| [MessageProvider](./MessageProvider.md) | 151 | 系統訊息 |
| [LogProvider](./LogProvider.md) | 405 | 日誌管理 |
| [TransProvider](./TransProvider.md) | 121 | 交易回寫設計 |
| [SolutionProvider](./SolutionProvider.md) | 77 | 方案管理 |
| [ScheduleProvider](./ScheduleProvider.md) | 94 | 排程管理 |
| [NotifyProvider](./NotifyProvider.md) | 35 | 系統公告（⚠️ 未完整實作） |
| [SysparasProvider](./SysparasProvider.md) | 28 | 系統參數管理 |
| [RecordlockProvider](./RecordlockProvider.md) | 49 | 記錄鎖定 |
| [GlobalProvider](./GlobalProvider.md) | 144 | 全域設定 |
| [GlobalFileProvider](./GlobalFileProvider.md) | 56 | 全域檔案 |
| [IndexProvider](./IndexProvider.md) | 63 | 首頁 |
| [ProcessorProvider](./ProcessorProvider.md) | 104 | 處理器 |
| [ITableProvider](./ITableProvider.md) | 111 | iTable 設計 |

## Adapter — 介面卡

| 介面卡 | 行數 | 說明 |
|--------|------|------|
| [Flow](./Flow_Adapter.md) | 1040 | 流程引擎介面卡 |
| [FlowParser](./FlowParser_Adapter.md) | 1302 | 流程定義解析器（JSON/XML 互轉） |
| [RWDPageParser](./RWDPageParser_Adapter.md) | 1103 | RWD 頁面解析器（模組 JSON 產生） |
| [ITableParser](./ITableParser_Adapter.md) | 1526 | iTable 頁面解析器 |
| [TransformType](./TransformType_Adapter.md) | 758 | Bootstrap 3→5 轉換器 |
| [Processor](./Processor_Adapter.md) | 702 | 處理器介面卡（PROC 呼叫） |
| [Parser](./Parser_Adapter.md) | 204 | 報表解析介面卡 |
| [Redis](./Redis_Adapter.md) | 123 | Redis 快取介面卡 |

## Extension — 擴充方法

| 檔案 | 行數 | 說明 |
|------|------|------|
| [IFormCollectionExtensions](./IFormCollectionExtensions.md) | 159 | 表單參數擴充（ToQueryOptions/ToUpdateOptions 等） |
| [SessionExtensions](./SessionExtensions.md) | 48 | Session 擴充（Get/SetClientInfo） |
| [HttpContextExtensions](./HttpContextExtensions.md) | 29 | HttpContext 擴充 |

## 其他

| 檔案 | 行數 | 說明 |
|------|------|------|
| [MSBuild](./MSBuild.md) | 239 | 模組編譯工具 |

## 架構流程

```
HTTP Request
  → Controller（WebClient）
  → BaseProvider.CreateProvider(type)
    → AccountProvider    ← type="account"
    → DataProvider       ← type="data"
    → SecurityProvider   ← type="security"
    → MenuProvider       ← type="design" + mode
    → FileProvider       ← type="file"
    → ChatProvider       ← type="chat"
    → ...
  → Provider.ProcessRequest(param)
    → DataModule.GetDataset() / UpdateDataset() / CallMethod()
  → HTTP Response
```
