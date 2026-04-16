# DataProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/DataProvider.cs` |
| 行數 | 1040 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `BaseProvider` |

## 用途

運行時的資料操作中心，負責前端與後端資料模組之間的橋接。處理查詢（GetDataset）、更新（UpdateDataset）、方法呼叫（CallMethod）、選單載入、記錄鎖定、LINE Notify 整合、UserDef 報表管理，以及 ChatUX AI 對話功能。

## ProcessRequest 模式

| mode | 方法 | 說明 |
|------|------|------|
| `getMenus` | `GetMenus()` | 取得使用者選單樹（runtimeMenu） |
| `getMenusFavor` | `GetMenusFavors()` | 取得使用者收藏選單清單 |
| `getDataset` | `GetDataset(module, command, options)` | 查詢資料集 |
| `updateDataset` | `UpdateDataset(module, command, datas, options)` | 更新資料集（新增/修改/刪除） |
| `callMethod` | `CallMethod(module, method, parameters)` | 呼叫 Server 元件方法 |
| `callProcessorMethod` | `CallProcessorMethod(id, parameters)` | 呼叫 Processor 元件方法 |
| `getConfig` | `GetConfig()` | 取得系統組態 |
| `getTitle` | `GetUserLocaleItem(mode, name)` | 取得多語系標題文字 |
| `loadHtml` | `LoadHtml(page, panelID, loadScript)` | 載入 RWD 頁面 HTML |
| `setClientInfo` | `SetClientInfo(key, value)` | 設定 ClientInfo 屬性並更新 Session |
| `checkSession` | `CheckSession()` | 檢查 Session 是否有效 |
| `downloadData` | `DownloadData(id, isUpdate, caches)` | 離線下載：渲染 RWD 頁面 + 取得所有資料集 |
| `addLock` | `AddLock(module, command, row, lockType)` | 新增記錄鎖定 |
| `removeLock` | `RemoveLock(module, command, rows)` | 移除記錄鎖定 |
| `ssoSolution` | `SsoSolution(solution)` | 切換至其他方案（SSO） |
| `lineNotify` | `LineNotify(name, code, url, state)` | LINE Notify 設定與綁定 |
| `saveMenusFavor` | (inline) | 儲存使用者收藏選單 |
| `addUserDefFile` | `AddUserDefFile(id, filePath, type, param)` | 新增使用者自訂報表（Word/Excel） |
| `addUserDefTable` | `AddUserDefTable(id, type, remoteName, rFormat, description, status)` | 新增/更新使用者自訂表單定義 |
| `getUserDefTable` | `GetUserDefTable(id, type)` | 取得使用者自訂表單定義 |
| `getFileExist` | `GetFileExist(fileName)` | 檢查檔案是否存在（含 backup） |
| `addUserDefReport` | `AddUserDefReport(id, report)` | 儲存使用者自訂報表 JSON |
| `getUserDefReport` | `GetUserDefReport(id)` | 讀取使用者自訂報表 JSON |
| `deleteReport` | `DeleteReport(id)` | 刪除使用者自訂報表 |
| `getReportExist` | `GetReportExist(id)` | 檢查報表是否存在 |
| `getColumnDefs` | `GetColumnDefs(remoteName)` | 取得欄位定義（含 schema） |
| `chatUXGetForm` | `chatUXGetForm(param)` | ChatUX AI 功能：解析自然語言指令找出對應表單 |
| `chatUXGetColumn` | `chatUXGetColumn(param)` | ChatUX AI 功能：欄位名稱匹配 |
| `chatUXGetAccess` | `chatUXGetAccess(param)` | ChatUX AI 功能：檢查使用者是否有權限 |

## 關鍵方法

### 核心資料操作

| 方法 | 說明 |
|------|------|
| `GetDataset(module, command, options)` | 透過 `DataModule` 查詢資料，SystemTable 命令自動切換為 `DatabaseType.Split`，結果以 JSON 回傳，含日誌紀錄 |
| `UpdateDataset(module, command, datas, options)` | 透過 `DataModule` 更新資料，自動記錄成功/失敗日誌 |
| `CallMethod(module, method, parameters)` | 呼叫 Server 端方法，結果序列化回傳 |
| `CallProcessorMethod(id, parameters)` | 委派給 `ProcessorProvider` 處理 |

### 選單

| 方法 | 說明 |
|------|------|
| `GetMenus()` | 從 `runtimeMenu` 取得選單，依據語系選擇 CAPTION 欄位，產生樹狀結構 |
| `GetMenusFavors()` | 從 `runtimeMenuFavor` 取得使用者收藏 |
| `GetCaptionName(locale)` | 語系對應 CAPTION 欄位名（en → CAPTION0, zh-tw → CAPTION1, zh-cn → CAPTION2 等） |

### 記錄鎖定

| 方法 | 說明 |
|------|------|
| `AddLock(module, command, row, lockType)` | 根據 `config.recordLock` 決定使用記憶體鎖（`LockHelper`）或資料庫鎖（`DBLockHelper`） |
| `RemoveLock(module, command, rows)` | 批次移除記錄鎖定 |
| `GetLockName(database, commandText, keys, row)` | 組合鎖定名稱格式：`{database}.{tableName}({key=value,...})` |

### LINE Notify

`LineNotify` 方法支援以下子操作（由 `name` 參數控制）：

| name | 說明 |
|------|------|
| `getLineConfig` | 取得 LINE Notify Client ID/Secret 設定 |
| `getGroupID` | 取得使用者所屬群組（非角色） |
| `setLineToken` | 用 OAuth code 換取 token 並寫入 USERS.LINE 或 USERGROUPS.LINE |
| `getClientInfo` | 回傳 ClientInfo |

### UserDef 報表

| 方法 | 說明 |
|------|------|
| `AddUserDefFile(id, filePath, type, param)` | 解析 Word/Excel 匯入檔，辨識欄位型別，寫入 SYS_USERDEF，產生 coldef |
| `AddUserDefTable(id, type, remoteName, ...)` | 新增/更新 SYS_USERDEF 自訂表單定義 |
| `AddUserDefReport(id, report)` | 將報表 JSON 寫入 `design/report/{solution}/{id}.json` |

### ChatUX AI

| 方法 | 說明 |
|------|------|
| `chatUXGetForm(param)` | 先透過 `ChatProvider` 解析自然語言取得命令/功能名稱，再從 MENUTABLE 比對表單，找不到時進行第二輪 AI 匹配 |
| `chatUXGetColumn(param)` | 透過 `ChatProvider` 匹配欄位名稱 |
| `chatUXGetAccess(param)` | 檢查使用者群組是否在 `config.chatUXAccess` 允許清單中 |

## 備註

- 所有 `GetDataset` / `UpdateDataset` / `CallMethod` 呼叫皆透過 `LogHelper` 記錄操作日誌（Normal / Error / Unknown）。
- `SystemTable` 模組的命令（除 `coldef` 外）自動使用 `DatabaseType.Split`。
- `DownloadData` 用於離線模式，一次取得 RWD 頁面 HTML 與所有關聯資料集。
- 記錄鎖定支援「記憶體鎖」與「資料庫鎖」兩種模式，由 `config.recordLock` 設定決定。
