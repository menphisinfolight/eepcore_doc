# COLDEF

## 用途

**欄位定義中繼資料表**（Column Definition Metadata Table）。

COLDEF 是 EEP Core 的核心系統表之一，用來儲存各「資料表 + 欄位」的顯示屬性與 UI 控制設定。所有模組頁面的欄位標題、編輯遮罩、下拉選單、必填驗證、報表顯示等元資料，都從這張表讀取，而非寫死在程式碼中。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **動態頁面生成** | 前端、後端根據 COLDEF 動態渲染資料表的欄位（標題、格式、必填等） |
| **Word / Excel 報表** | `ParserHelper`、`WordProvider` 查詢 COLDEF 取得欄位 CAPTION 對應，產生表頭 |
| **自動補填** | `ColdefProvider.AddColdef()` 掃描資料表結構，自動新增缺少的 COLDEF 記錄 |
| **多語系標題** | CAPTION ~ CAPTION8 對應不同語系的欄位顯示名稱 |
| **查詢模式** | `QUERYMODE` 控制該欄位在查詢條件列的表現形式 |
| **下拉選單** | `DD_NAME` 關聯下拉資料來源（如 SYS_REFVAL） |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/ColdefProvider.cs` | 提供 `load`/`add`/`remove`/`save` 四種模式的操作 API |
| `EEPServerTools.Core/Utility/ParserHelper.cs` | 報表系統讀取 COLDEF 取得欄位標題對應 |
| `EEPBase.Core/Utility/DatabaseHelper.cs` | `GetColdefTable()` 查詢指定 TABLE_NAME 的所有 COLDEF 記錄 |
| `SystemTable.Core/UserModule.cs` | `ucColdefForWord_onBeforeApply()`、`ucSYS_DOCFILES_onBeforeDelete()` 等鉤子 |
| 前端 `bootstrap.infolight.nodeflow.js` | 流程通知中使用 COLDEF 替換欄位代碼為顯示標題 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **TABLE_NAME** | `nvarchar(200)` PK | NOT NULL | 所屬資料表名稱 |
| **FIELD_NAME** | `nvarchar(200)` PK | NOT NULL | 欄位名稱 |
| **SEQ** | `decimal(12,0)` | NULL | 排序序號，決定欄位在頁面/報表的顯示順序 |
| **FIELD_TYPE** | `nvarchar(20)` | NULL | 欄位類型，如 `GF`(群組標題)、`GH`(表頭)、`R`(唯讀)、`C`(可編輯) 等 |
| **IS_KEY** | `nvarchar(1)` | NOT NULL | 是否為 Key 欄位 (`Y`/`N`) |
| **FIELD_LENGTH** | `decimal(12,0)` | NULL | 欄位長度上限 |
| **FIELD_SCALE** | `decimal(12,0)` | NULL | 小數位數（針對 numeric/decimal 類型） |
| **CAPTION** | `nvarchar(40)` | NULL | 預設語言的欄位顯示標題（最重要） |
| **CAPTION1~8** | `nvarchar(40)` | NULL | 多語系標題 1~8（8 個欄位） |
| **EDITMASK** | `nvarchar(10)` | NULL | 編輯時的格式遮罩（如 `999-0000` 電話格式） |
| **NEEDBOX** | `nvarchar(13)` | NULL | UI 控制項類型，如 `DateBox`、`TextBox`、`NumberBox`、`ComboBox` 等 |
| **CANREPORT** | `nvarchar(1)` | NULL | 是否可用於報表 (`Y`/`N`) |
| **CANEDIT** | `nvarchar(1)` | NULL | 是否可編輯 (`Y`/`N`，後期擴充) |
| **EXT_MENUID** | `nvarchar(20)` | NULL | 外部程式連結（點擊欄位時開啟指定的 MENUID 頁面） |
| **DD_NAME** | `nvarchar(40)` | NULL | 下拉選單關聯的資料字典名稱（對應 SYS_REFVAL.REFVAL_NO 或 SYS_PARAS.COLUMNNAME） |
| **DEFAULT_VALUE** | `nvarchar(1024)` | NULL | 欄位預設值 |
| **CHECK_NULL** | `nvarchar(1)` | NULL | 是否強制不可為空 (`Y`/`N`) |
| **QUERYMODE** | `nvarchar(20)` | NULL | 查詢列的表現模式（如 `normal`、`range`） |
| **FIELD_MERGE** | `nvarchar(1)` | NULL | 是否在報表中合併儲存格 (`Y`/`N`，後期擴充) |
| **FIELD_VISIBLE** | `nvarchar(1)` | NULL | 欄位是否可見 (`Y`/`N`，後期擴充) |
| **FIELD_VALIDATE** | `nvarchar(40)` | NULL | 驗證規則名稱 |
| **FIELD_TOTAL** | `nvarchar(1)` | NULL | 是否在報表中加總 (`Y`/`N`，後期擴充) |
| **FIELD_READONLY** | `nvarchar(1)` | NULL | 是否唯讀 (`Y`/`N`，後期擴充) |
| **SHOWAPP** | `nvarchar(1)` | NULL | 是否在 APP 顯示 (`Y`/`N`，後期擴充) |
| **GROUPTITLE** | `nvarchar(30)` | NULL | 群組標題（後期擴充） |
| **DESCRIPTION** | `nvarchar(200)` | NULL | 欄位詳細說明（後期擴充） |

### 跨資料庫差異

#### 主要型別對照（CREATE TABLE 新裝）

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `TABLE_NAME` | `nvarchar(200)` | `varchar2(200)` | `nvarchar(200)` | ⚠️ `NVARCHAR(20)` | `NVARCHAR(50)` |
| `FIELD_NAME` | `nvarchar(200)` | `varchar2(200)` | `nvarchar(200)` | ⚠️ `NVARCHAR(20)` | `NVARCHAR(50)` |
| `DEFAULT_VALUE` | `nvarchar(1024)` | `varchar2(1024)` | `nvarchar(1024)` | `NVARCHAR(1024)` | `LVARCHAR(1024)` |
| 其他 char 類欄位 | `nvarchar(n)` | `varchar2(n)` | `nvarchar(n)` | `NVARCHAR(n)` | `NVARCHAR(n)` |
| 數值欄位（SEQ / FIELD_LENGTH / FIELD_SCALE） | `decimal(12,0)` | `decimal(12,0)` | `decimal(12,0)` | `DECIMAL(12,0)` | `DECIMAL(12,0)` |

> ⚠️ **DB2 / Informix 的 PK 欄位長度大幅短於其他 DB**：
> - DB2：`TABLE_NAME` / `FIELD_NAME` 僅 **20 字元**
> - Informix：50 字元
> - MSSQL / Oracle / MySQL：200 字元
>
> 若實際使用中有表名或欄位名超過此長度，DB2 / Informix 會寫入失敗或截斷。建議 DBA 手動 `ALTER` 拉長（MSSQL 建表腳本有 `ALTER COLUMN ... nvarchar(200)` 的升級補丁，但 Oracle / MySQL / DB2 / Informix **沒有對應的 ALTER 補丁**）。

#### SP7 升級補欄位支援矩陣（舊版升級時是否自動 ALTER ADD）

新裝環境下五個 DB 的 CREATE TABLE 都有完整 36 個欄位，**無差異**。真正差異在**從舊版升級 SP7 時**，建表腳本是否含 `ALTER TABLE ADD` 自動補欄位：

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `CANEDIT` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_MERGE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_VISIBLE` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_TOTAL` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `FIELD_READONLY` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `SHOWAPP` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `DESCRIPTION` | ✅ | ✅ | ❌ | ❌ | ❌ |
| `GROUPTITLE` | ✅ | ❌ | ❌ | ❌ | ❌ |
| **PK 長度升級**（TABLE_NAME / FIELD_NAME 拉到 200） | ✅ | ❌ | ❌ | ❌ | ❌ |

> 來源：`EEPWebClient.Core/wwwroot/sql/{sql,oracle,mysql,db2,informix}/systemTables.sql` 的 ALTER TABLE 段落。

**結論**：
- **MSSQL**：升級自動化最完整
- **Oracle**：補了 6 個欄位，**仍缺 `FIELD_READONLY` / `GROUPTITLE` 與 PK 長度升級**
- **MySQL / DB2 / Informix**：建表腳本**完全沒有 ALTER 補欄位邏輯**。客戶若從舊版升 SP7，必須手動 `ALTER TABLE COLDEF ADD ...` 補所有 8 個欄位

#### 手動補欄位 SQL 範本

給 DBA 在 MySQL / DB2 / Informix 升級時參考（逐條檢查是否已存在；Oracle 還要補 `FIELD_READONLY` 和 `GROUPTITLE`）：

```sql
-- MySQL
ALTER TABLE COLDEF ADD CANEDIT nvarchar(1) NULL;
ALTER TABLE COLDEF ADD FIELD_MERGE nvarchar(1) NULL;
ALTER TABLE COLDEF ADD FIELD_VISIBLE nvarchar(1) NULL;
ALTER TABLE COLDEF ADD FIELD_TOTAL nvarchar(1) NULL;
ALTER TABLE COLDEF ADD FIELD_READONLY nvarchar(1) NULL;
ALTER TABLE COLDEF ADD SHOWAPP nvarchar(1) NULL;
ALTER TABLE COLDEF ADD DESCRIPTION nvarchar(200) NULL;
ALTER TABLE COLDEF ADD GROUPTITLE nvarchar(30) NULL;

-- Oracle（CREATE TABLE 已含 FIELD_READONLY / GROUPTITLE，舊表升級缺這兩個要補）
ALTER TABLE COLDEF ADD FIELD_READONLY varchar2(1) NULL;
ALTER TABLE COLDEF ADD GROUPTITLE varchar2(30) NULL;

-- DB2
ALTER TABLE COLDEF ADD COLUMN CANEDIT NVARCHAR(1);
ALTER TABLE COLDEF ADD COLUMN FIELD_MERGE NVARCHAR(1);
-- ... 其餘同列，型別 NVARCHAR(1)，DESCRIPTION NVARCHAR(200)，GROUPTITLE NVARCHAR(30)

-- Informix
ALTER TABLE COLDEF ADD CANEDIT NVARCHAR(1);
ALTER TABLE COLDEF ADD FIELD_MERGE NVARCHAR(1);
-- ... 其餘同列

-- DB2 / Informix 若表名超過 20 / 50 字元還要先拉長 PK 欄位（型別與 CREATE 同）
-- DB2：ALTER TABLE COLDEF ALTER COLUMN TABLE_NAME SET DATA TYPE NVARCHAR(200);
-- Informix：ALTER TABLE "COLDEF" MODIFY ("TABLE_NAME" NVARCHAR(200) NOT NULL);
```

---

## 主鍵

```
PRIMARY KEY (TABLE_NAME, FIELD_NAME)
```

同一張資料表的同一個欄位，在 COLDEF 中只會有一筆記錄。

---

## 備註

- COLDEF 是設計階段的元資料，**不是** Runtime 執行的 SQL Schema。
- 當使用者透過設計工具（如 Systablecopy）建立或修改模組時，系統會自動寫入 / 更新這張表。
- 刪除 `SYS_DOCFILES`（報表定義）時，會一併刪除對應的 COLDEF 記錄，確保報表與欄位定義一致性。
