
# EEP 設計工具端點 API 參考

> 給 EEP 設計工具 CLI 化用的完整端點地圖。原始碼來源：`EEPWebClient.Core/Controllers/{Account,Design,UserDesign}Controller.cs` + 各 Provider。


## 目錄

1. 認證流程
2. `/Account` — 登入 / 工作階段
3. `/Design` — 設計操作（依 `type` 分流）
4. `/UserDesign` — 文件 / Word / Excel 操作
5. `/UserDesign/Add`、`/Upload`、`/DownloadFile` — 檔案上傳下載
6. 參數格式特殊說明
7. CLI 完整範例
8. 回傳格式總表
9. 系統資料表對照
10. 認證 / Dev 模式規則


## 1. 認證流程

```
POST /Account (mode=logon, dev=true) → LogonSuccess + Set-Cookie
              ↓
        requests.Session 帶 cookie
              ↓
   /Design / /UserDesign 全部走這條
   (ClientInfo.Dev == true 才能呼叫，否則丟 "Timeout")
```

**所有 /Design 與 /UserDesign 端點都檢查 `ClientInfo.Dev == true`**（除了 `/Design type=database mode=saveAdmin` 是首次初始化用）。

工作階段相關 cookie：`devdatabase`、`devsolution`（保存 7 天）


## 2. `/Account`（POST/GET）

走 `AccountProvider.ProcessRequest(IFormCollection)` 分流。

| Mode | 用途 | 必要參數 | 選擇參數 | 回傳 |
|---|---|---|---|---|
| `logon` | 登入 | `user`、`password`、`database`、`solution` | `dev=true`、`captcha` | ClientInfo + Set-Cookie |
| `logoff` | 登出 | — | — | Empty |
| `refreshLogon` | 工作階段 keep-alive | — | — | 更新 ClientInfo |
| `getDatabases` | 列可用 DB | — | — | `[{name}]` |
| `getSolutions` | 列 Solution | — | — | 從 `MENUITEMTYPE` 取 |
| `getSolutionVisible` | UI 是否顯示 Solution 選單 | — | — | `{solutionVisible: bool}` |
| `getDBVisible` | UI 是否顯示 DB 選單 | — | — | `{dbVisible: bool}` |
| `getCaptcha` | 取 CAPTCHA 圖片 | — | — | binary（如 `logonValidate=true`）|
| `resetP` | 申請重設密碼 | `user`、`database`、`email` | — | 確認訊息 |
| `changeP` | 改密碼（已登入） | `opassword`、`password` | — | OK / error |
| `registerU` | 註冊新帳號 | `user`、`database`、`userName`、`password`、`email` | — | 確認訊息（須 `registerUser=true`） |
| `checkVerify` | 檢查 MAUI 授權 | — | — | `{MAUI_License: "Y"\|"N"}` |
| `switch` | 切換身份（dev only） | `user` | — | 新工作階段 |
| `getBiometric` | 是否支援生物辨識 | — | — | `{biometric: bool}` |
| `getLogonUrl` | 取登入頁 URL（SSO） | — | — | URL 字串 |


## 3. `/Design`（POST）

主要設計操作端點。`DesignProvider.CreateProvider(HttpContext, type)` factory 依 `type` 分派到 24+ 個專用 Provider。

### 3.1 `type=table` — 資料表操作

`TableProvider`

| Mode | 用途 | 主要參數 | 回傳 / 備註 |
|---|---|---|---|
| `load` | 讀資料 | `table`、`keys`（逗號分隔）、`QueryOptions` | `{rows, total}` 分頁；keys 啟用 SelectPaging |
| `save` | 新增/修改/刪除資料列 | `rows={table, inserted, updated, deleted}` | 三段式 diff 格式 |
| `loadDataTypes` | 取 SQL 資料型別清單 | — | `[{value}]`（DB 方言相關）|
| `loadSchema` | 取表 schema | `table` | `[{name, dataType, allowNull, key}]` |
| `changeSchema` | 建/改/刪表 | `table`、`columns`(JSON)、`keys`、`changeType=create\|alter\|drop` | columns 是 `{inserted, updated, deleted}` 三段式 |
| `exportexcel` | 匯出 .xlsx | `table`、`ExportOptions` | 檔名 `YYYYMMDDHHmmssfff.xlsx`，存 `/design/export/` |
| `deleteAll` | 清空表 | `table` | 用 DB 方言 `DELETE ALL` |
| `export` | 匯出 schema+data | `values=[表名]`（JSON）、`includeData=true` | 產 `.sqlx`：`{tableName, tableSchema, insertStrings}` |
| `getColumns` | 取欄位（給 Word/Excel 編輯器） | `tableName` | `[{FIELD_NAME, CAPTION}]` |
| `importSampleOrg` | 匯入範例組織 | — | 預建組織階層 |
| `batchUpdate` | 批次更新欄位 | `tableName`、`toValue`、`fromValue`、`valueType=field\|value` | 影響筆數；`field` 直接 `SET col=col2`、`value` 包字串 |

### 3.2 `type=schema` — Schema 查詢

`SchemaProvider`

| Mode | 用途 | 主要參數 | 回傳 / 備註 |
|---|---|---|---|
| `getTable` | 列表 / 取欄位 | `database`、`id`（表名，可空）、`filter`、`withColdef` | 樹狀結構；id 空則列表 |
| `getColumn` | 取 SQL 結果欄位 | `database`、`commandText`、`table`、`remoteName`、`used`、`autoParam` | `[{value, text, editor}]`，支援 SAP RFC |
| `viewData` | 執行 SQL 取資料 | `database`、`commandText`、`QueryOptions`、`autoParam` | `{rows, total}` |
| `getCommand` | 列 InfoCommand 模組 | `id`（模組名） | 樹狀；id 空則列模組 |
| `getDetailCommand` | 取 master-detail | `module`、`command` | `[{value}]` |
| `query` | 執行 mutation SQL | `database`、`commandText` | `ExecuteNonQuery` 影響筆數（INSERT/UPDATE/DELETE/DDL）|

### 3.3 `type=menu` — 選單管理（含 RWD/Server CS）

`MenuProvider`。**Server CS 與 RWD JSON 共用此端點**，靠 `menuType` 區分。

`menuType` 可選值：`server`、`bootstrap`、`netflow`、`report`、`chat`、`itable`、`word`、`excel`、`trans`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `get` | 載入選單樹 | `id`（資料夾）、`filter` | 樹狀；id 為資料夾名（`table`、`coldef`、`server_g`、`bootstrap_g`...） |
| `getGlobalMenus` | 全域搜尋 | `value` | `[{MODULE, NAME, CONTENT, editable}]` |
| `getTemplate` | 列模板 | `id`（模板資料夾） | 從 `/wwwroot/template/{folder}/` 讀 .json + .desc |
| `getJSTemplate` | 列 JS 片段 | `id`（menuType） | `[{id, text, content}]` 從 `/wwwroot/template/{menuType}_js/` |
| `load` | 載單一選單 | `id`、`menuType`、`template` | 完整 JSON，從 `design/menu/{menuType}/{solution}/{id}.json` |
| `loadTemplate` | 載模板 JSON | `id`（路徑：`template/bootstrap/Name`） | 從 `/wwwroot/{id}.json` |
| `save` | 儲存選單 | `id`、`menuType`、`template`、`datas`(JSON) | 寫 `design/menu/{menuType}/{solution}/{id}.json` |
| `add` | 新建選單 | `menuType`、`menuGroup`、`text`(menu id)、`datas`、`overwrite` | **Server CS**：`menuType=server`；**RWD**：`menuType=bootstrap` |
| `copy` | 複製選單 | `id`、`menuType`、`menuGroup`、`text`(新名)、`overwrite` | — |
| `rename` | 改名選單 | `id`、`menuType`、`menuGroup`、`text`、`overwrite` | 內部 flag rename=true |
| `remove` | 刪選單 | `id`、`menuType`、`template` | — |
| `check` | 驗證 JSON | `id`、`menuType` | 錯誤訊息或空 |
| `checkCapacity` | 檢查空間用量 | — | 磁碟使用統計 |
| `buildAll` | 重建所有選單 | — | 編譯為 runtime JS/JSON |
| `getLocalizableValue` | 取 i18n | `id`、`menuType`、`refresh` | 樹狀可翻譯字串 |
| `saveLocalizableValue` | 存 i18n | `id`、`menuType`、`datas` | — |
| `getSecurityValue` | 取選單 ACL | `id`、`menuType`、`user`、`group`、`activity`、`refresh` | 用戶/群組/動作矩陣 |
| `copySecurityValue` | 批次複製 ACL | `id`、`menuType`、`secType=user\|group\|activity`、`value`、`values` | — |
| `saveSecurityValue` | 存 ACL | `id`、`menuType`、`users`、`groups`、`activities`、`datas` | 複雜矩陣 |
| `getSecurityResult` | 計算有效權限 | `id`、`menuType`、`user` | — |
| `getSecurityActivities` | 列可用動作 | `id`、`menuType` | `[活動字串]` |
| `getSecurityMode` | 取 ACL 模式 | `id`、`menuType` | 集中 vs 分散 |
| `saveSecurityMode` | 設 ACL 模式 | `id`、`menuType`、`secMode` | — |
| `getSSOAddress` | 取 SSO 連結 | `user`、`parameters`、`id`、`menuType` | URL 編碼選單存取 |
| `getNonLogonAddress` | 取免登入連結 | `parameters`、`id`、`menuType` | 公開連結 |
| `addToNonlogon` | 開放免登入 | `id`、`menuType` | — |
| `removeNonlogon` | 撤除免登入 | `id`、`menuType` | — |
| `getByDatabase` / `getBySolution` | 多租戶過濾 | `IFormCollection` | — |
| `exportVersion` | 匯出版本 | `IFormCollection` | zip 檔名 |
| `getMethod` | 取方法簽章 | `id`（模組/命令）、`proc` | 從編譯後 components |
| `exportSystemfile` | 匯出系統選單 | `values`(JSON 陣列)、`menuType` | — |
| `export` | 匯出選單 | `values`(JSON id 陣列)、`menuType` | 批次匯出 |
| `exportProj` / `loadProj` | 匯出 / 匯入專案 | `id`、`script` | IDE 整合 |
| `validateScript` | 檢查 JS/SQL | `script` | 錯誤訊息或空 |

### 3.4 `type=coldef` — 資料字典

`ColdefProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `load` | 取欄位定義 | `command=coldef\|coldefForWord`、`QueryOptions` | 從 `SYS_COLDEF` 分頁 |
| `add` | 自動產生 coldef | `table` | 從 schema + 命名規則推 caption / 型別 |
| `remove` | 刪除欄位定義 | `table` | 刪該表所有 COLDEF |
| `save` | 存欄位定義 | `datas`(JSON 陣列) | 寫 `SYS_COLDEF` |

### 3.5 `type=security` — 安全 / 使用者管理

`SecurityProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `load` | 取安全資料 | `command=user\|group\|role\|menu\|org\|SYS_CARDS`、`QueryOptions` | 不同 command 回不同樹 |
| `save` | 更新安全資料 | `command`、`datas`、`UpdateOptions` | 寫 ACL / 用戶資料 |
| `renameSolution` | 改 Solution 名 | `oldName`、`newName` | 連動更新 `MENUITEMTYPE`、`MENUTABLE`、`MENUFAVOR` |
| `importAccount` | 匯入 LDAP/AD | `domain`、`user`、`password`、`importMode=merge\|replace` | 寫 `SYS_USERS`、`SYS_GROUPS` |
| `getOnlineUsers` | 列在線用戶 | — | `[{user, session}]` |
| `removeSession` | 強制登出 | `id`（session id） | — |
| `exportLog` / `exportSqlLog` / `exportUser` / `exportGroup` / `exportUserAccess` / `exportGroupUser` / `exportTreeNode` / `exportOrg` | 各種 Excel 匯出 | 各 mode 自有參數 | 回檔名 |

### 3.6 `type=processor` — 處理器除錯

`ProcessorProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `callDebugMethod` | 呼叫處理器 | `parameter={ProcessorID, Parameter:{}}` | 設計階段除錯 |
| `previewService` | 預覽服務呼叫 | `data`(JSON) | dry-run |

### 3.7 `type=control` — 元件屬性 / 圖示 / 檔案

`ControlProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `getProperty` | 取元件屬性 | `category=bootstrap\|components\|netflow\|report\|itable`、`controlType`、`propertyType` | reflection 取屬性陣列 |
| `getEditor` | 取 editor 類型 | `category` | RWD 控制項用 |
| `getValueType` | 取 value type | `category`、`controlType` | 巢狀型別 |
| `getToolItem` | 取工具列項目 | `category`、`controlType` | 動態 UI |
| `getIcon` | 取圖示庫 | `category=fontawesome\|bootstrap` | `[{value, text}]` CSS class |
| `getHtml` | 渲染 HTML | `value` | preview/validate |
| `getFile` / `getUserFile` / `removeUserFile` | 取/列/刪客製 CSS、JS | `id`、`fileType=css\|js`、`fileName` | `design/css`、`design/js` |
| `getComponent` | 取元件定義 | `id`（模組）、`componentType` | 從 server 模組 |

### 3.8 `type=database` — DB 連線管理

`DatabaseProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `get` | 列 DB | — | `[{name, selected}]` 含當前選擇 |
| `set` | 切 DB | `value` | 設 cookie + session |
| `load` | 取 DB 設定 | — | 樹狀 JSON |
| `save` | 存 DB 設定 | `datas`(JSON 陣列) | 寫 config |
| `test` | 測連線 | `conn_str`、`pwd`、`connType=mssql\|mysql\|postgresql` | — |
| `createDB` | 建 DB | `conn_str`、`dbName`、`connType` | DDL `CREATE DATABASE` |
| `createSystemTable` | 建系統表 | `name`（DB 名，預設 ClientInfo.Database） | 建 `SYS_*` 系列 |
| `settings` | 取 DB 設定 | — | 方言相關 |
| `getSystemDatabase` | 取系統 DB 名 | — | `SYS_*` 所在 |
| `deploy` / `export` | 部署 / 匯出 schema | `from`、`to`、`pages`、`includeData` | 跨環境同步 |
| `compare` | schema diff | `from`、`to`、`pages`、`includeData` | 產 diff SQL |
| `sp` / `view` | 列 SP / View | — | DB metadata |
| `getView` | 取 SP/View 定義 | `name`、`viewMode=sp\|view` | SQL 字串 |
| `createViewSP` / `saveViewSP` / `deleteViewSP` | DDL CREATE/ALTER/DROP | `name`、`ctype=sp\|view`、`sql`、`param` | — |

### 3.9 `type=solution` — Solution 管理

`SolutionProvider`

| Mode | 用途 | 主要參數 |
|---|---|---|
| `get` | 列 Solution | — |
| `load` | 取 Solution 資料 | `command`、`QueryOptions` |
| `save` | 更新 | `command`、`datas`、`UpdateOptions` |
| `set` | 切 Solution | `value` |

### 3.10 `type=global` — 全域設定

`GlobalProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `load` | 取全域設定 | — | 含 publicKey |
| `save` | 存全域設定 | `data`(JSON) | — |
| `loadParam` / `saveParam` | 取/存自訂參數 | `data` | — |
| `changeKey` | 輪換加密金鑰 | — | — |
| `setLogo` / `resetLogo` | 上傳/重設 logo | `logoType=header\|menu\|...` | 存 `design/images` |
| `testSSOUrl` | 驗 SSO 連線 | — | 連線測試 |
| `listChatModels` | 列 AI 模型 | `chatType`、`chatBaseUrl`、`chatAuthHeader`、`chatAuthPrefix`、`chatKey` | HTTP GET `/models`，timeout 15s。`{ok, models, url}` 或 `{ok:false, error}` |
| `getAbbyyInfo` | OCR 授權 | — | ABBYY |
| `getMapType` | 地圖供應商 | — | GIS |

### 3.11 其他 type 一覽

| Type | Provider | 主要 mode |
|---|---|---|
| `notify` | NotifyProvider | `load`、`remove` — `SYS_NOTIFY` |
| `log` | LogProvider | `load`(分 0-7 等級 + getChatLogs/getServerLogs/getSqlLogs)、`save`、`view`、`viewSql` |
| `versionControl` | VersionControlProvider | `load`、`get`、`remove`、`rollback`、`editDesc` |
| `index` | IndexProvider | `load`、`create`、`drop`（全文搜尋）|
| `font` | FontProvider | `get`、`save`、`reset` |
| `schedule` | ScheduleProvider | `load`、`save`、`logs`、`clearLogs` — `SYS_SCHEDULE` |
| `chat` | ChatProvider | `send`、`chatTest`、+ 20+ AI 輔助設計模式 |
| `menuGroup` | MenuGroupProvider | `load`、`add`、`set`、`remove`、`rename` |
| `globalFile` | GlobalFileProvider | `get`、`save` |
| `message` | MessageProvider | `get`、`save`（i18n）|
| `recordlock` | RecordlockProvider | `get`、`release` |
| `netflow` | NetFlowProvider | `load`、`save`、`reset` — `SYS_NETFLOW` |
| `getFlowText` / `totalFlowAnalysis` / `timeEffectivenessAnalysisOfProcess` / `processBottleneckAnalysis` | FlowAnalysisProvider | 流程分析 |
| `fileList` | FileListProvider | `get` — 列 design/files |
| `sysparas` | SysparasProvider | `save` — `SYS_PARAS` |

未支援的 type 會丟例外：`"DesignProvider.{type} not supported"`


## 4. `/UserDesign`（POST）

`UserDesignProvider.CreateProvider(HttpContext, type)` factory，支援 type：`menu`、`word`、`excel`、`trans`、`table`、`itable`。

### 4.1 `type=table` — 表查詢輔助

`UserTableProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `loadSchema` | 取表 schema | `table` | 同 /Design table loadSchema |
| `getTableNames` | 列已知表名 | — | 含 `SYS_DOCFILES`、`SYS_XLSFILES` |
| `getColumnNames` | 取欄位（含 key flag） | `table` | 給下拉選單用 |
| `getTableDefinations` | 多表定義查詢 | `tables={tableName: [captions]}` | 跨 SYS_DOCFILES、SYS_XLSFILES 等 |
| `getColumnTypes` | 取欄位型別清單 | — | DB 方言相關 |
| `getMoveDefinations` | 取資料搬移定義 | `table` | — |
| `getUserDict` | 查使用者字典 | `value` | 速查 |

### 4.2 `type=word` — Word 設計

`WordProvider`

| Mode | 用途 | 主要參數 | 備註 |
|---|---|---|---|
| `load` | 取 Word 設計 | `id`（檔名不含 .docx） | `{id, info, coldefs, codes, setting}`，解析 .docx 推欄位 |
| `getColumnNames` | 取欄位 | `table` | 給欄位對應 UI |
| `save` | 存 Word 設計 | `id`、`datas` | 存設定（不是 .docx 本身）|
| `remove` | 刪 Word | `id` | 從 `SYS_DOCFILES` 移除 |
| `write` | 產 Word 檔 | `id`、`datas`、`exportFile` | 產 .docx；`exportFile=true` 直接回檔 |

### 4.3 `type=excel` — Excel 設計

`ExcelProvider`，介面同 word，副檔名換成 .xlsx，從 `SYS_XLSFILES`。

| Mode | 用途 | 主要參數 |
|---|---|---|
| `load` | 取 Excel 設計 | `id` |
| `save` | 存設計 | `id`、`datas` |
| `remove` | 刪 Excel | `id` |
| `write` | 產 .xlsx | `id`、`datas`、`exportFile` |

### 4.4 `type=trans` — 交易模板

`TransProvider`，操作 `SYS_TRSFILES`，介面：`load`、`save`、`remove`。

### 4.5 `type=menu` / `type=itable`

- `menu` → 轉派到 `/Design` 的 `MenuProvider`（同 mode 集合）
- `itable` → `ITableProvider`（互動式表格）


## 5. 檔案上傳 / 下載

### 5.1 `/UserDesign/Add`（POST，multipart）

匯入 Word/Excel 設計

| 參數 | 類型 | 用途 | 備註 |
|---|---|---|---|
| `Files[0]` | 檔案 | 要匯入的文件 | 限 .doc / .docx / .xls / .xlsx |
| `newRule` | bool | 產生新 coldef 規則 | true 時先備份再覆蓋 |

回傳：成功 `{error: null}` / 失敗 `{error: "..."}`

副檔判型：`.doc/.docx` → type=word、`.xls/.xlsx` → type=excel。Word 自動備份到 `design/doc/{solution}/backup/`。

### 5.2 `/UserDesign/Upload`（POST，multipart）

僅儲存檔案（不處理）

| 參數 | 類型 | 備註 |
|---|---|---|
| `Files[0]` | 檔案 | .doc/.docx/.xls/.xlsx |

回傳成功：`[{name, size}]`，失敗：`{error}`

### 5.3 `/UserDesign/DownloadFile`（GET）

下載產出檔案（Word / Excel 匯出等）

| 參數 | 類型 | 備註 |
|---|---|---|
| `fileType` | string | 副檔名（如 `docx`、`xlsx`） |
| `fileName` | string | 不含副檔的檔名 |

實作：先找 `design/doc/{solution}/backup/`、再找主資料夾；從 backup 來的檔名後加 `_下載`。回傳含 `Content-Disposition: attachment` 的 binary。

無工作階段檢查（client 取自家產出的）。


## 6. 參數格式特殊說明

### 6.1 `keys`（逗號分隔字串）

```
keys = "ID"                    # 單一主鍵
keys = "ID,Seq"                # 複合主鍵（逗號、無空白）
```

用於 table load / export 啟用 SelectPaging。

### 6.2 `columns`（changeSchema 三段式 diff）

```json
{
  "inserted": [
    {"name": "NewCol", "dataType": "nvarchar", "allowNull": true, "key": false}
  ],
  "updated": [
    {"name": "OldCol", "dataType": "int"}
  ],
  "deleted": [
    {"name": "OldCol2"}
  ]
}
```

`dataType` 是**原始 SQL 字串**（例如 `INT IDENTITY(1,1)`、`NVARCHAR(255)`）— server 直接拼進 ALTER TABLE，沒有進一步 parse。

### 6.3 `datas`（依 mode 不同）

- **table save**：`{table, inserted, updated, deleted}`
- **menu save**：完整選單 JSON 物件
- **security save**：`{table:"command", inserted, updated, deleted}`
- **coldef save**：JSON 陣列（每個元素一個欄位定義）

### 6.4 `QueryOptions`（標準分頁）

```json
{
  "startIndex": 0,
  "pageSize": 20,
  "total": true,
  "whereItems": [],
  "orderItems": []
}
```

### 6.5 `ExportOptions`

```json
{
  "startIndex": 0,
  "pageSize": 100,
  "keys": "ID",
  "count": 1000
}
```


## 7. CLI 完整使用範例（Python）

```python
import requests, json

BASE = "http://localhost"
session = requests.Session()

# 1. 登入（取得 cookie）
r = session.post(f"{BASE}/Account", data={
    "mode": "logon",
    "user": "001",
    "password": "pwd",
    "database": "SQL105",
    "solution": "SOLUTION1",
    "dev": "true"
})
assert "LogonSuccess" in r.text

# 2. 建表
session.post(f"{BASE}/Design", data={
    "type": "table",
    "mode": "changeSchema",
    "table": "DEMO_TABLE",
    "changeType": "create",
    "keys": "ID",
    "columns": json.dumps({
        "inserted": [
            {"name": "ID", "dataType": "INT IDENTITY(1,1)", "allowNull": False, "key": True},
            {"name": "Name", "dataType": "NVARCHAR(255)", "allowNull": True, "key": False},
        ],
        "updated": [], "deleted": []
    })
})

# 3. 驗證 schema
r = session.post(f"{BASE}/UserDesign", data={
    "type": "table", "mode": "loadSchema", "table": "DEMO_TABLE"
})
print(r.json())  # [{"name":"ID","dataType":"int identity",...}]

# 4. 產生資料字典
session.post(f"{BASE}/Design", data={
    "type": "coldef", "mode": "add", "table": "DEMO_TABLE"
})

# 5. 部署 Server CS 選單
with open("my_module.json") as f:
    menu_def = f.read()
session.post(f"{BASE}/Design", data={
    "type": "menu", "mode": "add",
    "menuType": "server",
    "menuGroup": "公用",
    "text": "MY_MODULE",
    "datas": menu_def,
    "overwrite": "true"
})

# 6. 部署 RWD JSON
session.post(f"{BASE}/Design", data={
    "type": "menu", "mode": "add",
    "menuType": "bootstrap",
    "menuGroup": "公用",
    "text": "MY_FORM",
    "datas": json.dumps({...rwd_def...}),
    "overwrite": "true"
})

# 7. 產 Word
r = session.post(f"{BASE}/UserDesign", data={
    "type": "word", "mode": "write",
    "id": "MyTemplate",
    "datas": json.dumps({...payload...}),
    "exportFile": "true"
})
filename = r.text  # 例 "20260509103000.docx"

# 8. 下載 Word
r = session.get(f"{BASE}/UserDesign/DownloadFile",
    params={"fileType": "docx", "fileName": filename.replace(".docx", "")})
with open("output.docx", "wb") as f: f.write(r.content)

# 9. 匯出多張表 schema + data
r = session.post(f"{BASE}/Design", data={
    "type": "table", "mode": "export",
    "values": json.dumps(["DEMO_TABLE", "OTHER_TABLE"]),
    "includeData": "true"
})
sqlx_name = r.text

# 10. 比較兩 DB schema
r = session.post(f"{BASE}/Design", data={
    "type": "database", "mode": "compare",
    "from": "SQL105_DEV",
    "to": "SQL105_UAT",
    "pages": json.dumps(["table", "menu"]),
    "includeData": "false"
})

# 11. 登出
session.post(f"{BASE}/Account", data={"mode": "logoff"})
```


## 8. 回傳格式總表

### 分頁資料

```json
{ "rows": [...], "total": 100 }
```

### 樹狀結構（選單 / 模組 / 群組）

```json
[
  {
    "id": "folder_id",
    "text": "Display Name",
    "iconCls": "icon-class",
    "state": "closed",
    "attributes": {...},
    "checked": false
  }
]
```

### 錯誤回應

任何例外回 HTTP 200 + 例外訊息純文字。生產環境不夾 stack trace。

### 檔案下載

binary stream + `Content-Disposition: attachment; filename=...`

### loadSchema 結果

```json
[
  { "name": "ID",   "dataType": "int identity", "allowNull": "0", "key": true },
  { "name": "Name", "dataType": "nvarchar(255)", "allowNull": "1", "key": false }
]
```


## 9. 系統資料表對照

| 表 | 操作 Provider | 用途 |
|---|---|---|
| `SYS_COLDEF` | ColdefProvider | 資料字典：欄位名稱、型別、驗證 |
| `SYS_DOCFILES` | WordProvider | Word 設計 metadata |
| `SYS_XLSFILES` | ExcelProvider | Excel 設計 metadata |
| `SYS_TRSFILES` | TransProvider | 交易模板 metadata |
| `SYS_USERS` | SecurityProvider | 使用者帳號 |
| `SYS_GROUPS` | SecurityProvider | 群組 / 角色 |
| `SYS_ROLES` | SecurityProvider | 角色定義 |
| `SYS_PARAS` | GlobalProvider | 系統參數 |
| `SYS_LOG_*` | LogProvider | 稽核日誌 |
| `MENUITEMTYPE` | AccountProvider | Solution / 租戶 |
| `SYS_MENU` | SecurityProvider | 選單樹 |
| `SYS_ORG` | SecurityProvider | 組織結構 |
| `SYS_CARDS` | SecurityProvider | 權限卡 |
| `SYS_NOTIFY` | NotifyProvider | 通知 |
| `SYS_SCHEDULE` | ScheduleProvider | 排程任務 |
| `SYS_NETFLOW` | NetFlowProvider | 工作流定義 |
| `SYS_RECORDLOCK` | RecordlockProvider | 鎖定中的紀錄 |


## 10. 認證 / Dev 模式規則

1. **登入時帶 `dev=true`** → ClientInfo.Dev = true，工作階段才能呼叫 /Design 與 /UserDesign
2. **/Design 與 /UserDesign 全部端點檢查**：
   ```csharp
   if (clientInfo == null || !clientInfo.Dev)
       throw new EEPException("Timeout");
   ```
3. **例外**：`/Design type=database mode=saveAdmin` 用於首次初始化，不檢查工作階段，但要 admin key
4. **/Account 端點**混合：`logon`、`getDatabases` 不需工作階段；`changeP`、`switch`、`logoff` 需要
5. **/UserDesign/DownloadFile** 寬鬆：只檢查 `clientInfo != null`（不要求 Dev）

工作階段 cookie：`devdatabase`、`devsolution`，保存 7 天。


## 11. 檔案系統佈局（產出物落地點）

```
design/
├── doc/{solution}/{id}.docx        # Word 設計
├── doc/{solution}/{id}.xlsx        # Excel 設計
├── doc/{solution}/backup/          # 自動備份
├── export/                         # 匯出暫存（YYYYMMDDHHmmssfff.xlsx / .sqlx）
├── config/                         # admin.txt、menu_group.json
├── files/                          # 上傳資產
├── images/                         # 選單 / logo 圖示
├── css/                            # 客製 CSS
├── js/                             # 客製 JS
├── svg/                            # SVG
├── geojson/                        # GIS
├── menu/
│   ├── server/{solution}/          # Server CS
│   ├── bootstrap/{solution}/       # RWD JSON
│   ├── netflow/{solution}/         # 工作流
│   ├── report/{solution}/          # 報表
│   └── ...
└── log/                            # 應用日誌

wwwroot/
├── template/
│   ├── bootstrap/                  # RWD 模板
│   ├── bootstrap_js/               # RWD JS 片段
│   └── ...
└── json/keywords.json              # 型別推斷規則
```


## 12. 與 GUI 對應

設計工具的 GUI 操作幾乎都 1:1 對應到上述端點。CLI 化只是把 GUI 的點擊路徑改成 HTTP 呼叫。

| GUI 操作 | 對應端點 |
|---|---|
| 開設計工具登入 | `/Account mode=logon dev=true` |
| 左側「資料表」→ 建表 | `/Design type=table mode=changeSchema changeType=create` |
| 「資料字典」→ 自動產生 | `/Design type=coldef mode=add` |
| 「Server」→ 新建模組 | `/Design type=menu menuType=server mode=add` |
| 「RWD」→ 新建 | `/Design type=menu menuType=bootstrap mode=add` |
| 「Word 設計」→ 新建 | `/UserDesign/Upload` + `/UserDesign type=word mode=save` |
| 「Word 設計」→ 產生文件 | `/UserDesign type=word mode=write` |
| 「資料庫」→ 比較 schema | `/Design type=database mode=compare` |
| 「選單」→ 編輯權限 | `/Design type=menu mode=getSecurityValue / saveSecurityValue` |


## 後續延伸（給 CLI 設計者參考）

1. **錯誤處理**：所有端點失敗都回 HTTP 200 + 純文字訊息，CLI 必須先檢查 body 是否為 `LogonSuccess` / 合法 JSON
2. **重試**：工作階段過期會回 `Timeout`，需要重新 logon
3. **檔案路徑**：所有 `id` 都可能會被當路徑 segment（特別是 menu、word、excel），注意不要含 `/`、`..`、`\\`
4. **Solution 切換**：跨 solution 操作需先 `/Design type=solution mode=set`，影響後續所有 menu、word、excel 路徑
5. **Database 切換**：類似 solution，呼叫 `/Design type=database mode=set`
6. **批次操作**：menu、table 都有 `export` / `exportProj` 可一次匯出多個，CLI 可包裝 manifest YAML → 批次部署
7. **冪等性**：`menu add` 加 `overwrite=true` 變更新模式；`table changeSchema` 用 `alter` 而非 `create` 避免重建
8. **檔案下載**：產出後檔名要記下來（很多 mode 回傳就是檔名字串），用 `/UserDesign/DownloadFile` 拉檔
