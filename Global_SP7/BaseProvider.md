# BaseProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/BaseProvider.cs` |
| 行數 | 832 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |

## 用途

此檔案定義了 EEP Core Provider 架構的基礎類別與工廠模式分派機制。所有 Provider 都直接或間接繼承自 `BaseProvider`，並透過 `DesignProvider.CreateProvider` 或 `UserDesignProvider.CreateProvider` 靜態方法，根據前端傳入的 `param["type"]` 建立對應的 Provider 實例。

## 繼承階層

```
BaseProvider (abstract)
├── AccountProvider          ← 直接繼承
├── DataProvider             ← 直接繼承
├── DesignProvider (abstract)
│   ├── MenuProvider
│   ├── ControlProvider
│   ├── SchemaProvider
│   ├── DatabaseProvider
│   ├── TableProvider
│   ├── SolutionProvider
│   ├── ColdefProvider
│   ├── SecurityProvider
│   ├── ProcessorProvider
│   ├── LogProvider
│   ├── GlobalProvider
│   ├── NotifyProvider
│   ├── SysparasProvider
│   ├── RecordlockProvider
│   ├── ScheduleProvider
│   ├── MessageProvider
│   ├── GlobalFileProvider
│   ├── ChatProvider
│   ├── FlowAnalysisProvider
│   ├── NetFlowProvider
│   ├── MenuGroupProvider
│   ├── FileListProvider
│   ├── VersionControlProvider
│   ├── IndexProvider
│   ├── FontProvider
│   └── FileProvider (type = "report" 或 null)
└── UserDesignProvider (abstract)
    ├── MenuProvider
    ├── WordProvider
    ├── ExcelProvider
    ├── TransProvider
    ├── UserTableProvider
    └── ITableProvider
```

## 列舉型別

### FileType

| 值 | 對應根目錄 |
|----|-----------|
| `Files` | `design/files` |
| `Images` | `design/images` |
| `Css` | `design/css` |
| `Doc` | `design/doc` |
| `Export` | `design/export` |
| `Geojson` | `design/geojson` |
| `Svg` | `design/svg` |

## DesignProvider.CreateProvider 分派表

此靜態方法根據 `type` 參數建立對應的 Provider：

| type 值 | 建立的 Provider |
|---------|----------------|
| `"menu"` | `MenuProvider` |
| `"control"` | `ControlProvider` |
| `"schema"` | `SchemaProvider` |
| `"database"` | `DatabaseProvider` |
| `"table"` | `TableProvider` |
| `"solution"` | `SolutionProvider` |
| `"coldef"` | `ColdefProvider` |
| `"security"` | `SecurityProvider` |
| `"processor"` | `ProcessorProvider` |
| `"log"` | `LogProvider` |
| `"global"` | `GlobalProvider` |
| `"notify"` | `NotifyProvider` |
| `"sysparas"` | `SysparasProvider` |
| `"recordlock"` | `RecordlockProvider` |
| `"schedule"` | `ScheduleProvider` |
| `"message"` | `MessageProvider` |
| `"globalFile"` | `GlobalFileProvider` |
| `"chat"` | `ChatProvider` |
| `"getFlowText"` / `"totalFlowAnalysis"` / `"timeEffectivenessAnalysisOfProcess"` / `"processBottleneckAnalysis"` | `FlowAnalysisProvider` |
| `"netflow"` | `NetFlowProvider` |
| `"menuGroup"` | `MenuGroupProvider` |
| `"fileList"` | `FileListProvider` |
| `"versionControl"` | `VersionControlProvider` |
| `"index"` | `IndexProvider` |
| `"font"` | `FontProvider` |
| `"report"` / `null` | `FileProvider` |

## UserDesignProvider.CreateProvider 分派表

| type 值 | 建立的 Provider |
|---------|----------------|
| `"menu"` | `MenuProvider` |
| `"word"` | `WordProvider` |
| `"excel"` | `ExcelProvider` |
| `"trans"` | `TransProvider` |
| `"table"` | `UserTableProvider` |
| `"itable"` | `ITableProvider` |

## 關鍵方法

### BaseProvider (abstract)

| 方法 | 說明 |
|------|------|
| `ProcessRequest(IFormCollection)` | 虛擬方法，子類別覆寫以處理 API 請求 |
| `ProcessFile(IFormCollection)` | 虛擬方法，子類別覆寫以處理檔案上傳 |
| `GetFolderPath(FileType, folder)` | 根據 FileType 取得檔案儲存路徑 |
| `GetMenuDir(menuType)` | 取得選單定義檔目錄：`design/{menuType}/{solution}` |
| `GetConfigPath(configName)` | 取得組態檔路徑：`design/config/{configName}` |
| `GetTemplateDir(menuType)` | 取得範本目錄：`wwwroot/template/{menuType}` |

### UserDesignProvider (abstract)

| 方法 | 說明 |
|------|------|
| `GetFilePath(fileName)` | 取得 `design/doc/{solution}/{fileName}` 路徑 |
| `GetBackupFilePath(fileName)` | 取得備份路徑 `design/doc/{solution}/backup/{fileName}` |
| `GetColumnNames(name)` | 取得資料表欄位名稱清單 |
| `GetSystemTableDefination(name)` | 取得系統表定義（USERS / GROUPS / ROLES） |
| `ToRecognizeWord(name, item, items, rules)` | 依據 `keywords.json` 規則辨識欄位型別（K/KN/KR/R/RV/T 等） |
| `GetRules()` | 讀取 `wwwroot/json/keywords.json` 取得辨識規則 |
| `GetTableInfos()` | 從 SYS_DOCFILES / SYS_XLSFILES 取得表單資訊 |
| `InsertRows / DeleteRows` | 透過 DataModule 新增/刪除資料 |

### TableDefination 類別

位於檔案底部的 POCO 類別，包含 `CommandName`、`TableName`、`Title`、`ValueField`、`TextField`、`ValueTitle`、`TextTitle`、`Columns` 屬性。

## 備註

- `BaseProvider` 和 `DesignProvider` / `UserDesignProvider` 均為 `abstract`，不能直接實例化。
- `DesignProvider` 用於設計端（Design API），`UserDesignProvider` 用於使用者端文件設計（Word/Excel/Trans）。
- 所有 Provider 的 `HttpContext` 與 `ClientInfo` 均在建構時或呼叫端設定。
- `ToRecognizeWord` 是 Word/Excel 匯入時自動辨識欄位型別的核心邏輯，規則定義在 `wwwroot/json/keywords.json`。
