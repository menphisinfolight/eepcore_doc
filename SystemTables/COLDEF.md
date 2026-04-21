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

Oracle 用 `varchar2`、其他 DB 用 `nvarchar` / `NVARCHAR`，數值欄位（`SEQ` / `FIELD_LENGTH` / `FIELD_SCALE`）全 DB 皆為 `DECIMAL(12,0)`。

⚠️ **`TABLE_NAME` / `FIELD_NAME` 長度跨 DB 差異大**：MSSQL / Oracle / MySQL 為 200，Informix 50，**DB2 僅 20**。實際表名超過 DB 欄位長度會寫入失敗或截斷。

👉 新裝缺欄位、SP7 升級 ALTER 支援矩陣、手動補欄位 SQL：**問題_SP7_跨資料庫欄位差異盤點**

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
