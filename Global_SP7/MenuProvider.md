# MenuProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/MenuProvider.cs` |
| 行數 | 2225 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` → `BaseProvider` |

## 用途

設計端最大的 Provider，負責 EEP Core 所有選單（Server / RWD / Workflow / Report 等）的 CRUD 操作、範本管理、版本控制整合、安全性（Security）設定、多語系（Locale）設定、SSO 網址產生、匯出/匯入、佈署（Deploy），以及 JavaScript 驗證。

## ProcessRequest 模式

### 選單 CRUD

| mode | 方法 | 說明 |
|------|------|------|
| `get` | `GetMenus(id, filter)` | 取得選單樹（資料表/欄位/元件/文件節點） |
| `getGlobalMenus` | `GetGlobalMenus(value)` | 全域搜尋所有選單中包含指定文字的項目 |
| `getTemplate` | `GetTemplates(id)` | 取得範本清單 |
| `getJSTemplate` | `GetJSTemplates(id)` | 取得 JS 範本清單與內容 |
| `load` | `GetMenu(id, template, menuType)` | 載入選單定義（JSON + 控制項型別 + script + desc + lock） |
| `loadTemplate` | `LoadTemplate(id)` | 載入範本 JSON |
| `save` | `SaveMenu(id, template, menuType, datas)` | 儲存選單（含版本控制提交、netflow XML 產生、Server 端 DLL 編譯） |
| `buildAll` | `BuildAll()` | 重新編譯所有 Server 端元件 DLL |
| `add` | `AddMenu(menuType, menuGroup, text, datas, overwrite)` | 新增選單（含合併/鎖定屬性處理） |
| `copy` | `CopyMenu(...)` | 複製選單 |
| `rename` | `CopyMenu(..., deleteSource=true)` | 重新命名選單（複製後刪除原始） |
| `remove` | `RemoveMenu(id, template, menuType)` | 刪除選單（.json/.js/.cs/.locale/.security/.desc/.lock/.xml） |
| `check` | `CheckMenu(id, menuType)` | 檢查選單/資料表/TRS 是否存在 |
| `checkCapacity` | `CheckCapacity()` | 容量檢查（目前固定回傳 true） |

### 多語系

| mode | 方法 | 說明 |
|------|------|------|
| `getLocalizableValue` | `GetLocalizableValue(id, menuType, refresh)` | 取得控制項多語系值（支援 en / zh-tw / zh-cn / zh-hk / ja-jp / ko-kr / it1 / it2） |
| `saveLocalizableValue` | `SaveLocalizableValue(id, menuType, datas)` | 儲存多語系值至 `.locale` 檔 |

### 安全性

| mode | 方法 | 說明 |
|------|------|------|
| `getSecurityValue` | `GetSecurityValue(id, menuType, user, group, activity, refresh)` | 取得指定使用者/群組/活動的安全性設定值 |
| `copySecurityValue` | `CopySecurityValue(id, menuType, secType, value, values)` | 複製安全性設定到其他使用者/群組 |
| `saveSecurityValue` | `SaveSecurityValue(id, menuType, users, groups, activities, datas)` | 儲存安全性設定至 `.security` 檔 |
| `getSecurityResult` | `GetSecurityResult(id, menuType, user)` | 計算指定使用者的最終安全性結果（使用者 + 群組合併） |
| `getSecurityActivities` | `GetSecurityActivities(id, menuType)` | 取得已設定安全性的活動清單 |
| `getSecurityMode` | `GetSecurityMode(id, menuType)` | 取得安全性模式（預設 "both"） |
| `saveSecurityMode` | `SaveSecurityMode(id, menuType, secMode)` | 儲存安全性模式 |

### SSO 與免登入

| mode | 方法 | 說明 |
|------|------|------|
| `getSSOAddress` | `GetSSOAddress(user, url, id, type, parameters)` | 產生 SSO 連結（含短網址） |
| `getNonLogonAddress` | `GetNonLogonAddress(url, id, type, parameters)` | 產生免登入連結（含短網址） |
| `addToNonlogon` | `AddToNonlogon(id, type)` | 將選單加入免登入 URL 清單（寫入 config.json） |
| `removeNonlogon` | `RemoveNonlogon(id, type)` | 從免登入 URL 清單移除 |

### 匯出與佈署

| mode | 方法 | 說明 |
|------|------|------|
| `getByDatabase` | `GetByDatabase(param)` | 切換資料庫後取得選單 |
| `getBySolution` | `GetBySolution(param)` | 切換方案後取得選單 |
| `exportVersion` | `ExportVersion(param)` | 匯出指定時間後的變更清單 |
| `getMethod` | `GetMethods(id, proc)` | 取得 Server 元件的方法清單（樹狀） |
| `exportSystemfile` | `ExportSystemfile(values, menuType)` | 匯出系統文件（表格 schema 或元件截圖 → DOCX） |
| `export` | `ExportMenu(values, menuType)` | 匯出選單為 ZIP（含 .json/.js/.cs/.locale/.security/.xml） |
| `exportProj` | `ExportProj(id, script)` | 編譯並匯出 Server 專案 DLL |
| `loadProj` | `LoadProj(id)` | 載入 Server 專案程式碼 |
| `validateScript` | `ValidateScript(script)` | 透過 JSLint 驗證 JavaScript |

## 關鍵方法

### 選單樹結構

| 方法 | 說明 |
|------|------|
| `GetMenus(folder, filter)` | 根目錄回傳 11 個資料夾節點（Table / DataDictionary / Server / RWD / Workflow / Report / ChatCoder / iTable / Word / Excel / TRS）；展開後根據類型載入不同內容 |
| `GetMenuNodes(type, group)` | 掃描 `design/{type}/{solution}/` 目錄的 `.json` 檔，支援 menu_group 分組 |
| `GetDocNodes(type, group)` | 從 SYS_DOCFILES / SYS_XLSFILES / SYS_TRSFILES / SYS_FILES 取得文件節點 |
| `FilterMenus(list, filter)` | 過濾選單節點（文字包含比對） |

### 選單群組

| 方法 | 說明 |
|------|------|
| `GetMenuGroup()` | 讀取 `design/menu_group.json` |
| `GetMenuGroupDic()` | 將群組 JSON 轉為 `{menuID: groupName}` 字典 |
| `SaveMenuGroup(groups)` | 儲存群組 JSON |
| `SetMenuGroup(text, type, group)` | 設定選單所屬群組 |

### 儲存與編譯

| 方法 | 說明 |
|------|------|
| `SaveMenu(id, isTemplate, menuType, datas)` | 儲存 JSON → 版本控制提交 → netflow 產生 XML → Server/Report 寫 .cs 並呼叫 `MSBuild.Build()` 編譯 DLL → Bootstrap 寫 .js → 儲存 .desc / .lock |
| `AddMenu(menuType, menuGroup, text, datas, overwrite)` | 新增時處理 Merge（合併屬性）與 Lock（鎖定屬性）邏輯，使用 `ControlHelper` 比對新舊控制項 |
| `BuildAll()` | 遍歷所有 Server JSON，重新編譯每一個 .cs 為 .Core.dll |

### 佈署

| 方法 | 說明 |
|------|------|
| `DeployMenu(from, to, pages, checkDate, isCreate, isVersion)` | 從來源方案佈署選單到目標方案，支援日期比對（只佈署較新的）與版本記錄（產生 ZIP） |
| `CopyMenu(id, menuType, group, text, options)` | 核心複製邏輯，支援覆寫、日期檢查、刪除來源、群組變更、版本紀錄 |

### 安全性檔案結構

`.security` 檔案為 JSON 格式：

```json
{
  "mode": "both",
  "users": { "admin": [...] },
  "groups": { "00": [...] },
  "activities": { "activity1": [...] }
}
```

每個項目的值為 JArray，包含 `{ name, value }` 物件。

### 控制項型別解析

| 方法 | 說明 |
|------|------|
| `GetControlTypes(typeItems, controls)` | 遞迴解析控制項樹，透過 `ControlHelper` 取得各型別屬性 |
| `GetFlowTypes()` | 取得所有 Workflow Activity 型別 |
| `GetReportTypes()` | 取得所有 Report Item 型別 |
| `GetITableTypes()` | 取得所有 iTable Item 型別 |
| `GetAssembly(category)` | 根據類別取得對應 Assembly（bootstrap → EEPRWDTools, server → EEPServerTools 等） |

## 備註

- 選單定義檔儲存在 `design/{menuType}/{solution}/` 目錄下，每個選單包含：`.json`（控制項定義）、`.js` 或 `.cs`（腳本）、`.desc`（描述）、`.lock`（鎖定欄位）、`.locale`（多語系）、`.security`（安全性）、`.xml`（netflow 專用）。
- `SaveMenu` 在 Server/Report 類型時會即時編譯 C# 程式碼為 DLL（`MSBuild.Build()`）。
- `AddMenu` 有合併邏輯：新增覆寫時，會保留原始檔案中標記為 Merge 或 Lock 的屬性值。
- 短網址服務使用 `pics.ee` API（與 AccountProvider 共用邏輯）。
- `ValidateScript` 使用外部 JSLint 工具驗證 JavaScript，透過 `.bat` 批次檔執行，並含 `CheckBatContent` 防止命令注入。
- `DeployMenu` 用於方案間的選單佈署，可產生版本差異紀錄（ZIP + TXT）。
- `CopyOptions` 類別定義了複製操作的選項：`NewID`、`Overwrite`、`CheckDate`、`IsCreate`、`DeleteSource`、`ChangeGroup`、`IsVersion`。
