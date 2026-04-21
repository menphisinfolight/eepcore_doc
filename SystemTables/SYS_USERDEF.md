# SYS_USERDEF

## 用途

**使用者自訂報表定義表**（User-Defined Report Definition）。

SYS_USERDEF 儲存使用者自訂的報表/匯出格式定義，支援多種類型（Word、Excel、Report、樞紐分析、統計圖表）。每筆記錄對應一個自訂報表範本，RFORMAT 欄位儲存報表的格式設定（JSON），GROUPCOLUMN 儲存群組欄位名稱。

### 使用場景

| 場景 | 說明 |
|------|------|
| **自訂報表儲存** | DataProvider 在使用者建立/修改自訂報表時 INSERT 或 UPDATE 定義記錄 |
| **自訂報表載入** | DataProvider.GetUserDefTable() 以 FILENAME + TYPE 查詢定義，取出 RFORMAT 作為格式設定 |
| **報表清單顯示** | 前端 ireport.js 的 userDefinedWord/userDefinedExcel/userDefined 等方法，以 datagrid 顯示 SYS_USERDEF 的記錄清單 |
| **remoteName 轉換** | `SYS_USERDEF_onBeforeExecuteSQL()` 將 whereStr 中的 `remoteName=模組.命令` 轉換為 `MASTERTABLE = 表名` |
| **工具列按鈕** | TableGrid/DataGrid 的工具列提供 UserDefined / UserPrint 按鈕觸發自訂報表功能 |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/DataProvider.cs` | 核心操作：InsertUserDefRows() / UpdateUserDefRows() / GetUserDefTable()，自訂報表的建立、更新、查詢 |
| `SystemTable.Core/UserModule.cs` | `SYS_USERDEF_onBeforeExecuteSQL()`：remoteName → MASTERTABLE 轉換 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `SYS_USERDEF` infocommand：`SELECT * FROM SYS_USERDEF`，keys: `FILENAME,TYPE`；`ucSYS_USERDEF` 有 `updateIfExists: true` |
| `EEPWebClient.Core/wwwroot/js/infolight/bootstrap.infolight.ireport.js` | 前端自訂報表 UI：userDefinedWord()、userDefinedExcel()、userDefined()（含樞紐分析、統計圖表） |
| `EEPRWDTools.Core/Controls/TableGrid.cs` | 工具列定義：UserDefined / UserPrint 按鈕 |
| `EEPRWDTools.Core/Controls/DataGrid.cs` | 同上 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FILENAME** | `nvarchar(20)` PK | NOT NULL | 報表檔案名稱（使用者定義） |
| **TYPE** | `nvarchar(20)` PK | NOT NULL | 報表類型（word / excel / report / pivot / chart） |
| **MASTERTABLE** | `nvarchar(20)` | NULL | 主檔資料表名稱（由 remoteName 的 CommandText 解析而來） |
| **DEFDATE** | `datetime` | NULL | 定義日期（自動填入 `yyyy-MM-dd hh:mm:ss`） |
| **OWNER** | `nvarchar(20)` | NULL | 擁有者（自動填入 ClientInfo.User） |
| **DESCRIPTION** | `nvarchar(20)` | NULL | 描述 |
| **GROUPCOLUMN** | `nvarchar(20)` | NULL | 群組欄位名稱（Word 匯出時的分組依據，後期擴充） |
| **RFORMAT** | `nvarchar(4000)` | NULL | 報表格式定義（JSON，儲存欄位順序、寬度、顯示設定等，後期擴充） |

### TYPE 報表類型

| 值 | 說明 | 對應前端方法 |
|----|------|-------------|
| `word` | Word 匯出格式 | `userDefinedWord()` |
| `excel` | Excel 匯出格式 | `userDefinedExcel()` |
| `report` | 列印報表格式 | `userDefined()` 中的 report 分頁 |
| `pivot` | 樞紐分析表 | `userDefined()` 中的 pivot 分頁 |
| `chart` | 統計圖表 | `userDefined()` 中的 chart 分頁 |

### 跨資料庫差異

#### 欄位存在度

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|:-:|:-:|:-:|:-:|:-:|
| `FILENAME` / `TYPE` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `MASTERTABLE` / `OWNER` / `DESCRIPTION` | ✅ | ✅ | ✅ | ✅ | ✅ |
| **日期欄位名** | `DEFDATE` | `DEFDATE` | `DEFDATE` | ⚠️ `` `DATE` `` | ⚠️ `DATE` |
| `GROUPCOLUMN` | ✅ | ✅ | ✅ | ✅ | ✅ |
| **`RFORMAT`** | ✅ | ✅ | ✅ | ❌ | ✅ |

> ⚠️ **日期欄位名跨 DB 不一致**：MSSQL / Oracle / MySQL 三個 DB 叫 `DEFDATE`，**Informix 叫 `DATE`、DB2 也叫 `` `DATE` `` 並用反引號**（`` ` ``，MySQL 語法 — 實際 DB2 不認反引號，腳本可能直接執行失敗）。讀這個欄位的程式需要做 DB 判斷。
>
> ⚠️ **DB2 缺 `RFORMAT` 欄位**（其他 4 DB 都有）。報表格式化功能在 DB2 無法運作。
>
> 兩個問題都像腳本撰寫誤植。建議 DBA 手動修正。

#### 型別對照

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `RFORMAT` | `nvarchar(4000)` | `varchar2(4000)` | `nvarchar(4000)` | ❌ 缺 | `LVARCHAR(4000)` |
| 日期欄位 | `datetime`（`DEFDATE`）| `date`（`DEFDATE`）| `datetime`（`DEFDATE`）| `datetime`（`` `DATE` ``）| `DATETIME YEAR TO SECOND`（`DATE`）|

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `GROUPCOLUMN` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `RFORMAT` | ✅ | ✅ | ❌ | ❌ | ❌ |

MSSQL / Oracle 都有 ALTER 升級補欄位邏輯；MySQL / Informix CREATE TABLE 已含 `GROUPCOLUMN` 與 `RFORMAT`，但升級腳本沒 ALTER；**DB2 CREATE TABLE 也缺 `RFORMAT`，且日期欄位名是 `` `DATE` `` 帶反引號**。

```sql
-- DB2（補 RFORMAT 並修正日期欄位名）
ALTER TABLE SYS_USERDEF ADD COLUMN RFORMAT NVARCHAR(4000);
ALTER TABLE SYS_USERDEF RENAME COLUMN "DATE" TO DEFDATE;  -- 先移除可能的反引號包裹

-- Informix（修正欄位名 DATE → DEFDATE）
ALTER TABLE "SYS_USERDEF" RENAME COLUMN "DATE" TO "DEFDATE";

-- MySQL（舊表升級時補 GROUPCOLUMN 與 RFORMAT）
ALTER TABLE SYS_USERDEF ADD GROUPCOLUMN nvarchar(20) NULL;
ALTER TABLE SYS_USERDEF ADD RFORMAT nvarchar(4000) NULL;
```

---

## 主鍵

```
PRIMARY KEY (FILENAME, TYPE)
```

---

## 資料生命週期

```
建立：使用者在 datagrid 工具列點擊 UserDefined
      → 選擇類型（Word/Excel/Report/Pivot/Chart）
      → 設定欄位、格式
      → DataProvider → InsertUserDefRows()
      → INSERT INTO SYS_USERDEF (FILENAME, TYPE, MASTERTABLE, DEFDATE, OWNER, DESCRIPTION, GROUPCOLUMN)

更新（含 RFORMAT）：使用者修改格式設定
      → DataProvider → UpdateUserDefRows()
      → UPDATE SYS_USERDEF SET RFORMAT=@rFormat, GROUPCOLUMN='', ...
      → ucSYS_USERDEF 有 updateIfExists: true，INSERT 時若存在則改為 UPDATE

載入：使用者選擇已存在的自訂報表
      → DataProvider.GetUserDefTable(id, type)
      → SELECT * FROM SYS_USERDEF WHERE FILENAME = @id AND TYPE = @type
      → 取出 RFORMAT（優先）或 GROUPCOLUMN 作為格式設定

查詢清單：前端顯示自訂報表列表
      → SYS_USERDEF infocommand
      → SYS_USERDEF_onBeforeExecuteSQL() 將 remoteName 轉為 MASTERTABLE 過濾
```

---

## 備註

- DEFDATE 欄位原名 `DATE`，後透過 `sp_rename` 改為 `DEFDATE`（SQL 腳本中有此操作）。
- GROUPCOLUMN 和 RFORMAT 為後期擴充欄位（SQL 腳本中有 ALTER TABLE 補欄位邏輯）。
- `updateIfExists: true` 使得 ucSYS_USERDEF 在 INSERT 時若主鍵已存在會自動改為 UPDATE。
- GetUserDefTable() 取 RFORMAT 時以 index [7] 和 [6] 存取（RFORMAT 是第 8 欄，GROUPCOLUMN 是第 7 欄），若 RFORMAT 為空則回退到 GROUPCOLUMN。
- 前端 datagrid 清單中顯示 FILENAME、DEFDATE、OWNER、DESCRIPTION 四個欄位，其中 DEFDATE 格式化為 `yyyy/MM/dd hh:mm:ss`。
