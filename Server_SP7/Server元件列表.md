# EEP Core Server 元件列表（SP7）

> 原始碼位置：`EEPServerTools.Core/`

## Components — 伺服器端元件

| 元件 | 繼承 | 行數 | 說明 |
|------|------|------|------|
| [ServerComponent](./ServerComponent.md) | Component | 32 | 伺服器元件基底類別 |
| [AutoNumber](./AutoNumber.md) | ServerComponent | 196 | 自動編號（搭配 SYSAUTONUM 表） |
| [InfoCommand](./InfoCommand.md) | SelectComp | 481 | 資料查詢命令（SELECT + 安全過濾 + 分頁） |
| [InfoDataSource](./InfoDataSource.md) | SelectComp | 228 | 主從資料來源（Master-Detail 關聯） |
| [InfoLine](./InfoLine.md) | SelectComp | 45 | 行資料來源 |
| [UpdateComponent](./UpdateComponent.md) | UpdateBaseComp | 620 | 資料更新元件（INSERT/UPDATE/DELETE + 事件鉤子） |
| [InfoTransaction](./InfoTransaction.md) | UpdateBaseComp | 679 | 交易回寫元件（搭配 SYS_TRSFILES） |
| [InfoMail](./InfoMail.md) | Component | 52 | 郵件發送元件 |
| [InfoPush](./InfoPush.md) | ServerComponent | 63 | 推播通知元件 |
| [InfoRecMail](./InfoRecMail.md) | ServerComponent | 126 | 郵件接收元件 |
| [InfoWarning](./InfoWarning.md) | ServerComponent | 365 | 預警/告警元件 |
| [ServerMove](./ServerMove.md) | ServerComponent | 406 | 資料搬移元件 |
| [Servicemanager](./Servicemanager.md) | — | 18 | 服務管理器（空殼） |

## DataModule — 資料模組核心

| 檔案 | 行數 | 說明 |
|------|------|------|
| [DataModule](./DataModule.md) | 1507 | 資料模組核心，載入 JSON 設定、管理元件、執行查詢/更新 |

## Utility — 工具類別

| 工具 | 行數 | 說明 |
|------|------|------|
| [EEPNetHelper](./EEPNetHelper.md) | 981 | 網路工具（HTTP 請求、LINE/Teams/釘釘推播） |
| [FlowHelper](./FlowHelper.md) | 851 | 流程引擎工具（啟動/簽核/查詢待辦） |
| [ParserHelper](./ParserHelper.md) | 969 | 報表解析工具（Word/Excel 欄位對應） |
| [ScheduleHelper](./ScheduleHelper.md) | 303 | 排程引擎（30 秒輪詢 SYS_SCHEDULE） |

## Adapter — 介面卡

| 介面卡 | 行數 | 說明 |
|--------|------|------|
| [Flow](./Flow.md) | 2032 | 流程引擎介面卡（IOrgInfoProvider、IAgentInfoProvider 等實作） |
| [Parser](./Parser.md) | 340 | 報表解析介面卡 |
| [Processor](./Processor.md) | 52 | 處理器介面卡 |
| [Redis](./Redis.md) | 123 | Redis 快取介面卡 |

## Enum — 列舉定義

| 檔案 | 行數 | 說明 |
|------|------|------|
| [Enum](./Enum.md) | 68 | DatabaseType、SecStyle 等列舉定義 |

---

## 元件繼承架構

```
Component（基底）
├── ServerComponent
│   ├── AutoNumber          — 自動編號
│   ├── InfoPush            — 推播通知
│   ├── InfoRecMail         — 郵件接收
│   ├── InfoWarning         — 預警/告警
│   └── ServerMove          — 資料搬移
├── SelectComp
│   ├── InfoCommand         — 資料查詢
│   ├── InfoDataSource      — 主從資料來源
│   └── InfoLine            — 行資料來源
├── UpdateBaseComp
│   ├── UpdateComponent     — 資料更新
│   └── InfoTransaction     — 交易回寫
└── InfoMail                — 郵件發送
```
