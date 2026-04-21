# MENUCHECKLOG

## 用途

**設計端元件版本管理歷史檔**（Version Control Archive）。

MENUCHECKLOG 儲存設計端元件（Server 模組 / Menu / Bootstrap / Client / Netflow / Processor / Report）**存檔前的舊版快照**。啟用版本管理時，每次使用者在 EEP 設計介面按下「存檔」，`VersionControlProvider.Submit()` 會比對磁碟上的舊內容與新內容，不同才把舊內容寫入此表。之後可以透過版本管理 UI 列表、預覽、回滾。

> 表名歷史沿用「CHECKLOG」（簽入檢查記錄）字樣，實際上 SP7 的運作是**版本存檔 archive**，與 `MENUTABLE.CHECKOUT` / `CHECKOUTDATE` 的「簽出鎖定」是另一套機制（未在本表參與運作）。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **存檔時自動備份** | `MenuProvider.SaveMenu` / `AddMenu` 呼叫 `VersionControlProvider.Submit`，在覆寫磁碟檔案前把舊版寫入此表 |
| **版本列表** | 設計端版本管理面板 → `POST /design/ type=versionControl, mode=load` → 回傳此元件所有歷史版本 |
| **版本預覽** | `mode=get` → 取出指定 LOGID 的 FILECONTENT，解析回 `controls` + `script` |
| **版本回滾** | `mode=rollback` → 先把當前磁碟版本 Submit 進表（作為 rollback 前的 backup），再把舊版 FILECONTENT 覆寫回磁碟 |
| **版本備註** | `mode=editDesc` → 改 `PACKAGE` 欄位 |

### 啟用條件

`design/config/global.cfg` 的 `versionControl` 必須為 `"true"` 才會啟用。關閉時 Submit 直接跳過寫入、Rollback 會丟 `Versiondisabled` 例外。

---

## 欄位結構

| 欄位名 | 資料類型（SQL Server） | 可為 NULL | 說明 |
|--------|----------------------|-----------|------|
| **LOGID** | `int IDENTITY(1,1)` PK | NOT NULL | 版本記錄識別碼 |
| **ITEMTYPE** | `nvarchar(20)` | NOT NULL | 方案識別碼（寫入時填入 `ClientInfo.Solution`，用於多方案隔離） |
| **PACKAGE** | `nvarchar(50)` | NOT NULL | 版本描述；存檔時常見值：`"overwrite backup"`（AddMenu）、`"rollback backup"`（Rollback）、使用者自訂 desc；可透過 editDesc 事後修改 |
| **PACKAGEDATE** | `datetime` | NULL | 此版本記錄的建立時間（舊版被存入 MENUCHECKLOG 的時刻）|
| **FILETYPE** | `nvarchar(10)` | NULL | 元件類型，實際值：`server` / `menu` / `bootstrap` / `client` / `netflow` / `processor` / `report` |
| **FILENAME** | `nvarchar(60)` | NULL | 元件 id（對應磁碟檔名，不含副檔名）|
| **FILEDATE** | `datetime` | NULL | **舊檔案的原始 mtime**（不是 PACKAGEDATE，而是被覆寫前磁碟上那份 `.json` 的 LastWriteTime）|
| **FILECONTENT** | `image` | NULL | 舊版完整內容（JSON 字串），包含 `controls`（設計時序列化的元件樹）+ `script`（C# / JS 原始碼） |
| **USERID** | `nvarchar(50)` | NULL | 觸發存檔的使用者帳號（**Oracle 建表腳本沒有此欄**，SQL Server 用 `ALTER TABLE ADD` 補上，見下方跨資料庫差異）|

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 | Informix |
|------|-----------|--------|-------|-----|----------|
| `LOGID` | `int IDENTITY(1,1)` | `integer NOT NULL` + 序列 | `int AUTO_INCREMENT` | `INT GENERATED ALWAYS AS IDENTITY` | `SERIAL NOT NULL` |
| `PACKAGEDATE` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `FILEDATE` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `FILECONTENT` | `image` | **`CLOB`** | `LONGBLOB` | `BLOB(10M)` | `BLOB`（+ 同名小寫欄位）|
| `USERID` | `nvarchar(50)`（後期 ALTER 補）| （無）| （無）| （無）| （無）|

> Oracle CLOB 每字面字串最多 4000 字元，Submit 因此把 FILECONTENT 切成 **3900 字元段**，以 ` to_clob` 連接子串（見 `VersionControlProvider.Submit` L226-233）。GetVersion 讀回時會 `content.Replace(" to_clob", "")` 還原、並針對 byte[] 型態（SQL Server image / MySQL LONGBLOB）用 `Encoding.UTF8.GetString` 轉回字串。

---

## 主鍵

```
PRIMARY KEY (LOGID)
```

---

## 相關元件 / InfoCommand

### 在 SystemTable.json 的 InfoCommand

| id | CommandText | 用途 |
|----|-------------|------|
| `menuLogView` | `SELECT LOGID, PACKAGE, PACKAGEDATE, FILEDATE, ITEMTYPE, FILETYPE, FILENAME, USERID FROM MENUCHECKLOG ORDER BY PACKAGEDATE DESC` | 版本列表（不含 FILECONTENT 避免一次拉太大）|
| `menuLog` | `SELECT * FROM MENUCHECKLOG` | 取單一版本（含 FILECONTENT）|
| `ucMenuLog` | UpdateComponent（`infocommand: menuLog`，`validateXss: false`）| 寫入 / 更新 / 刪除 |

### 相關核心程式碼

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/VersionControlProvider.cs` | 版本管理主邏輯：`Submit` / `GetVersions` / `GetVersion` / `EditDesc` / `RemoveVersions` / `Rollback` |
| `EEPGlobal.Core/Provider/MenuProvider.cs` | `AddMenu` L906、`SaveMenu` L986 在寫檔前呼叫 `VersionControlProvider.Submit` |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.versionControl.js` | 前端版本管理面板 |
| `EEPWebClient.Core/design/config/global.cfg` | `versionControl` 開關設定 |

---

## Submit 流程

```
存檔 / 新增元件
  → MenuProvider.SaveMenu 或 AddMenu
  → VersionControlProvider.Submit(id, type, datas)
    ├── 讀 global.cfg，確認 versionControl == "true"
    ├── 讀磁碟上舊的 {id}.json + {id}.cs/.js → 組成 oDatas
    ├── Compare(oDatas, datas)：
    │     controls JSON 字串 + script 字串都一樣 → 跳過
    ├── 不同才：
    │     切 3900 字元段（以 " to_clob" 連接）
    │     INSERT INTO MENUCHECKLOG (...)
    └── 回傳給呼叫者（但真正的存檔由 MenuProvider 繼續寫檔）
```

## Rollback 流程

```
POST /design/ type=versionControl, mode=rollback, id=..., menuType=..., version={LOGID}
  → VersionControlProvider.Rollback
    ├── 讀 global.cfg，確認 versionControl == "true"（否則丟 Versiondisabled）
    ├── GetVersion(id, type, version) 取出 MENUCHECKLOG 那筆的 FILECONTENT
    │     （version=0 是特殊值，代表「當前磁碟檔案」，一般 rollback 傳 LOGID）
    ├── Submit(id, type, { "desc": "rollback backup" })
    │     → 先把當前磁碟版本存成新的 MENUCHECKLOG 記錄（rollback 前 backup）
    ├── 用舊版 FILECONTENT 的 controls 覆寫 {id}.json
    ├── 用舊版 FILECONTENT 的 script 覆寫 {id}.js
    └── type=netflow 額外呼叫 Flow.SaveToXml 重建 .xml
```

---

## 資料生命週期

```
新建元件 (AddMenu) → 若磁碟已有舊檔 → Submit → INSERT MENUCHECKLOG
編輯存檔 (SaveMenu) → 比對無變化則跳過，有變化 → Submit → INSERT
回滾 (Rollback) → 先 Submit 當前版本（backup） → 再覆寫磁碟
手動管理 → 版本管理面板 → editDesc / remove 整列刪除
```

---

## 備註

- **FILETYPE 只有 7 種值**：`server` / `menu` / `bootstrap` / `client` / `netflow` / `processor` / `report`（見 `MenuProvider.SaveMenu` L984 允許清單）。非這些類型的存檔不進 MENUCHECKLOG。
- **FILECONTENT 儲存的是 JSON 字串**，邏輯把它當字串處理，即使 SQL Server 欄位型別是 `image`（byte[]），讀取時有 UTF8 decode 的 fallback。
- **版本管理和 MENUTABLE 的 CHECKOUT 欄位無關** — 原本文件提到的「簽入簽出流程」（MENUTABLE.CHECKOUT / CHECKOUTDATE）是另一套「選單簽出鎖」機制，SP7 的 VersionControlProvider 不會動那兩欄。
- **Oracle 沒有 USERID 欄位**，只有 SQL Server 有 ALTER TABLE 補欄位邏輯（`IF NOT EXISTS(select * from syscolumns...) ALTER TABLE`）。
- **清理建議**：長期累積的 FILECONTENT 會很佔空間，可定期清理老舊 LOGID（透過版本管理面板 `remove` mode 或直接 SQL 刪）。
- **停用版本管理**：`global.cfg` 設 `versionControl = "false"`，Submit 直接 no-op，Rollback 會丟例外。

👉 完整機制說明見 [版本管理機制](../其他(SP7)/版本管理機制.md)
