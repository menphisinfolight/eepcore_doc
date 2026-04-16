# RWDPage

> `EEPRWDTools.Core/RWDPage.cs` — 943 行

## 用途

**RWD 頁面渲染核心**（Page Renderer）。

RWDPage 是 EEPRWDTools.Core 模組的核心類別，繼承 `RWDContainerControl` 並實作 `IContainer`。負責讀取模組的 JSON 設定檔，將其轉換為控制項樹狀結構，並渲染為完整的 HTML 頁面（含 `<head>`、script/css 引用、`<body>`）。

每個 Bootstrap 前端頁面對應一個 RWDPage 實例。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `ClientInfo` | ClientInfo | 當前使用者連線資訊（User、Solution、Locale、Groups 等） |
| `Name` | string | 頁面名稱（對應 JSON 檔名，如模組名） |
| `Theme` | string | Bootstrap 主題名稱（預設 `"default"`） |
| `Offline` | bool | 離線模式（影響 script/css 路徑） |
| `LoadScript` | bool | 是否載入頁面專屬 JS 檔 |
| `HeadHtmls` | JArray | 額外注入 `<head>` 的 HTML 內容 |
| `Query` | Dictionary\<string, string\> | URL 查詢參數（用於安全驗證的 `p`、`pm`、`m`、`im` 等） |
| `rwdVersion` | string | RWD 版本號（來自 `RWDDef.RWD_Ver`） |
| `MenuInfo` | JObject | 選單權限資訊（含 secGroups、secUsers 的 CRUD 權限） |

## 建構函式

```csharp
RWDPage(ClientInfo clientInfo, string pageName, bool offline = false, bool loadScript = false)
RWDPage()  // 無參數建構
```

## JSON 檔案路徑規則

| 副檔名 | 路徑 | 用途 |
|--------|------|------|
| `.json` | `design/bootstrap/{Solution}/{Name}.json` | 頁面元件定義 |
| `.js` | `design/bootstrap/{Solution}/{Name}.js` | 頁面專屬 JavaScript |
| `.security` | `design/bootstrap/{Solution}/{Name}.security` | 安全性控制定義 |
| `.locale` | `design/bootstrap/{Solution}/{Name}.locale` | 多語系文字定義 |

## 核心方法

### 頁面渲染

| 方法 | 說明 |
|------|------|
| `Render()` → string | **主要入口**。呼叫 `LoadControls()` 載入控制項，再 `Render(html)` 產生完整 HTML |
| `Render(HtmlString html)` | 產生完整 HTML 文件（`<!DOCTYPE>` + `<html>` + header + body） |
| `RenderBody(string panelID)` | 僅渲染 body 部分。可指定 panelID 只渲染特定 Panel 內的控制項 |
| `RenderHtml(string value)` | 將任意 HTML 字串包裝成完整頁面（含 header） |
| `RenderIframe(string url)` | 產生 iframe 頁面（處理 IE 相容性、支援 `addTab`） |
| `RenderScript()` | 讀取並回傳頁面專屬 JS 檔內容 |

### 控制項載入

| 方法 | 說明 |
|------|------|
| `LoadControls()` | 核心載入流程（詳見下方） |
| `CreateComponent(JObject item)` | 依 `type` 反射建立控制項實例，呼叫 `LoadProperties` 載入屬性 |
| `LoadSecurity(JArray controls)` | 載入安全性設定，依使用者/群組/流程活動合併權限 |
| `LoadLocale(JArray controls)` | 載入多語系設定，依 locale 替換控制項文字 |

### 資源查詢

| 方法 | 說明 |
|------|------|
| `GetRemoteNames(string id)` | 遞迴掃描 JSON 控制項，收集所有 `remoteName`（資料來源名稱）及其父子關聯 |
| `MenuInfo` (getter) | 查詢系統表取得選單資訊、群組權限（allowAdd/Update/Delete）、使用者權限 |

## LoadControls 載入流程

```
LoadControls()
  ├─ 1. LoadScript → 讀取 {Name}.js，存入 HeadHtmls
  ├─ 2. 讀取 {Name}.json → JArray
  ├─ 3. LoadSecurity(items) → 合併安全性設定至控制項 JSON
  ├─ 4. LoadLocale(items) → 合併多語系設定至控制項 JSON
  ├─ 5. items.Select → CreateComponent() → 加入 Controls
  └─ 6. 安全性檢查（非 Dev 模式 + 有 Security 控制項時）
       ├─ 若有 p + pm 參數 → MD5 驗證通過則放行
       └─ 若 MenuInfo 無 id → 拋出 accessDenied 例外
```

## LoadSecurity 安全性合併邏輯

```
LoadSecurity(controls)
  ├─ 1. 讀取 {Name}.security 檔案
  ├─ 2. 取得 mode（"both" / "user"）
  ├─ 3. 查找當前 User 的安全性設定 → values
  ├─ 4. 遍歷 User 所屬 Groups 的安全性設定
  │    ├─ 若同名設定值衝突：
  │    │    ├─ mode="user" 且 User 有設定 → 保留 User 值
  │    │    └─ 否則 → 設為 mode 字串值
  │    └─ 若無衝突 → 直接合併
  ├─ 5. 若有 p + m 參數 → 檢查流程活動（activities）的安全性
  │    → MD5(p + ':' + activityKey) 比對 m 參數
  │    → 命中則覆寫對應安全性值
  └─ 6. ControlHelper.SetControlValues() → 將合併後的值套用到控制項 JSON
```

## RenderScripts 腳本載入邏輯

依據頁面中使用的控制項類型（`GetControlTypes()`），動態決定需要載入的 JS 檔案：

| 控制項 | 額外載入的 JS |
|--------|--------------|
| `Pivottable` | jQuery UI、D3、C3、pivot.js |
| `Tree` | bootstrap-treeview |
| `Card` | jquery.card |
| `Schedule` | bootstrap-calendar（含 locale） |
| `OrgChart` | jquery.orgchart |
| `Gantt` | dhtmlxgantt（含 locale、外掛） |
| `GeoChart` / `SvgChart` | echarts |
| `Logon` | jsencrypt（RSA 加密） |
| `TableGrid` | bootstrap-table（含 locale、fixed-columns） |
| `Map` / `Place` | 動態載入地圖 script |
| `Submenu` | bootstrap-submenu（v3）/ bootstrap-dropdown-hack（v5） |

基礎 JS 無論如何都會載入：jQuery、Bootstrap、datetimepicker、QRCode、Barcode、jSignature、fileinput、sweetalert、infolight 核心庫等。

## RenderStyleSheets 樣式載入邏輯

與 `RenderScripts` 類似，依控制項類型動態載入對應的 CSS。最後加上主題樣式：

```
bootstrap_{theme}.css  （theme 預設為 "default"）
bootstrap-responsive.css
global stylesheet
```

## MenuInfo 權限查詢

```
MenuInfo (getter)
  ├─ 1. 查詢 SystemTable.runtimeMenu → 找到 FORM == Name 的選單項
  │    （若有 im 參數 → 以 MD5(MENUID) 比對確認）
  ├─ 2. 以 MENUID 查詢 menuGroup → secGroups
  │    → 每個群組的 { allowAdd, allowUpdate, allowDelete }
  └─ 3. 以 MENUID 查詢 menuUser → secUsers
       → 每個使用者的 { allowAdd, allowUpdate, allowDelete }
```

## GetRemoteNames 資料來源收集

遞迴掃描頁面 JSON 中所有控制項的 `remoteName` 屬性：

```
GetRemoteNames(id)
  ├─ 讀取 {id}.json
  ├─ 遞迴 GetRemoteName(JObject) → 收集 remoteName + cacheName + parentObject
  └─ 若有 parentObject → 查找父控制項的 RemoteName → 填入 parentTable
```

## 備註

- RWDPage 同時實作 `IContainer`，因此可作為 `CreateComponent` 的容器，讓子控制項透過 `Container` 存取頁面層級資訊。
- 安全性驗證使用 MD5 雜湊比對（`Query["p"]` + `Query["pm"]`），非 Dev 模式且頁面含 `Security` 控制項時才啟用。
- `RenderIframe` 中包含 IE（ActiveXObject）相容處理，以 `window.open` 取代 iframe src 設定。
- Locale 對應有多層轉換：`GetLocale()` 將標準 locale（如 `zh-Hant-TW`）轉為簡寫（`zh-tw`），`GetControlLocale()` 再轉為各 JS 套件需要的格式（`zh-TW`）。
- `GetRemoteNames` 中存在重複呼叫 `GetRemoteName` 的冗餘程式碼（第一輪收集到 `remoteNames`，第二輪收集到 `controlNames` 但未使用 `controlNames`）。
- Bootstrap 版本影響範圍廣泛：除了 CSS class（由 `RWDDef` 處理），script 載入邏輯中也有 `rwdVersion == "5.3.3"` 的分支判斷（如 bootstrap-select 版本差異）。
