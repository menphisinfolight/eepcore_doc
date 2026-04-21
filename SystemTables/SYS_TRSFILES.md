# SYS_TRSFILES

## 用途

**資料表交易回寫定義表**（Table Transaction / Write-Back Definition）。

SYS_TRSFILES 定義 EEP Core 的「資料表交易」機制——當一張資料表（原始表）發生新增/修改/刪除時，自動觸發對另一張資料表（目的表）的欄位回寫。這是 EEP 的核心資料關聯功能，無需手寫程式碼即可實現跨表的資料同步。

### 核心概念

```
原始資料表（Source Table）── 交易定義 ──> 目的資料表（Target Table）

觸發時機（when）：
  I  = 新增時
  U  = 更改時
  D  = 刪除時
  IU = 新增更改時
  ID = 新增刪除時
  SI = 同步新增時

回寫方式（updateMode）：
  Increase        = 累加（+）
  Decrease        = 累減（-）
  DecreaseNotZero = 累減不能小於 0（-x）
  Replace         = 替換（=）
  ReplaceNegative = 替換為負值（=-）
  WriteBack       = 回寫（<=）
  WriteBackInc    = 回寫累加（<+）
  WriteBackDec    = 回寫累減（<-）

交易模式（transMode）：
  AutoAppend    = 自動新增（目的表無資料時自動 INSERT）
  Exception     = 例外（目的表無資料時拋出錯誤）
  Ignore        = 忽略（目的表無資料時跳過）
  AlwaysAppend  = 永遠新增（不管目的表有無資料都 INSERT）
  SyncAppend    = 同步新增
```

### 使用場景

| 場景 | 說明 |
|------|------|
| **庫存扣減** | 出貨單存檔 → 以出貨數量累減產品庫存（updateMode=Decrease） |
| **累計營收** | 訂單存檔 → 將金額累加到客戶的累計營收欄位（updateMode=Increase） |
| **關聯更新** | 出貨單聯絡人 → 更新到客戶資料表的聯絡人（updateMode=Replace） |
| **庫存異動日** | 出貨日期 → 回寫到產品的庫存異動日（updateMode=WriteBack） |

### 典型範例（出自原始碼 examples）

```
出貨單存檔時：
  - 將出貨金額累加到客戶資料表的累計營收欄位
  - 將聯絡人更新到客戶資料表的聯絡人
  - 關聯：出貨單.客戶編號 → 客戶資料表.客戶編號

出貨單明細存檔時：
  - 以出貨數量扣除產品資料表的現有庫存
  - 以出貨日期更換產品資料表的庫存異動日
  - 關聯：出貨單明細.產品編號 → 產品資料表.產品編號
```

### TRSFORMAT 結構

TRSFORMAT 欄位儲存 JSON 陣列，每個元素代表一條交易規則：

```json
[
  {
    "sourceTable": "出貨單",
    "targetTable": "客戶資料表",
    "when": "IU",
    "sourceField": "出貨金額",
    "targetField": "累計營收",
    "updateMode": "Increase",
    "keyFields": "客戶編號"
  },
  {
    "sourceTable": "出貨單",
    "targetTable": "客戶資料表",
    "when": "IU",
    "sourceField": "聯絡人",
    "targetField": "聯絡人",
    "updateMode": "Replace",
    "keyFields": "客戶編號"
  }
]
```

### 設計器

前端使用 `jquery.infolight.tableTransaction2.js` 提供視覺化設計器，在 EEP 設計工具的左側樹狀選單中，類型為 `trans`（顯示名稱 `TRS`），與 Word、Excel 同屬文件型別（docTypes）。

### 關聯表

```
SYS_TRSFILES.MASTERTABLE ──> 原始資料表
SYS_TRSFILES.DETAILTABLE ──> 原始明細表 1
SYS_TRSFILES.DETAIL2TABLE ──> 原始明細表 2
SYS_TRSFILES.DETAIL3TABLE ──> 原始明細表 3
```

TABLE_NAMES 陣列定義：`{ "Master", "Detail", "Detail2", "Detail3", "SubDetail" }`

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/TransProvider.cs` | 核心：load/save/remove 交易定義，解析 TRSFORMAT |
| `EEPGlobal.Core/Provider/MenuProvider.cs` | 設計工具：`trans` 類型的檔案列表、存在性檢查 |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.tableTransaction2.js` | 前端設計器：視覺化交易規則編輯 |
| `SystemTable.Core/UserModule.cs` | 鉤子：插入前先刪除同 TRSNAME 的舊記錄 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **TRSNAME** | `nvarchar(80)` PK | NOT NULL | 交易定義名稱 |
| **NAME** | `nvarchar(50)` | NULL | 顯示名稱 |
| **MASTERTABLE** | `nvarchar(50)` | NULL | 原始主檔資料表名稱 |
| **DETAILTABLE** | `nvarchar(50)` | NULL | 原始明細表 1 名稱 |
| **DETAIL2TABLE** | `nvarchar(50)` | NULL | 原始明細表 2 名稱（後期擴充） |
| **DETAIL3TABLE** | `nvarchar(50)` | NULL | 原始明細表 3 名稱（後期擴充） |
| **TRSFORMAT** | `nvarchar(max)` | NULL | 交易規則定義（JSON 陣列，見上方結構說明） |
| **TRSDATE** | `datetime` | NULL | 定義日期 |
| **OWNER** | `nvarchar(50)` | NULL | 擁有者 |

### 跨資料庫差異

五個 DB CREATE TABLE 都完整。⚠️ DB2 / Informix `TRSFORMAT` 上限 8000 字元。

👉 升級 ALTER 支援矩陣、手動補欄位 SQL：**問題_SP7_跨資料庫欄位差異盤點**

---

## 主鍵

```
PRIMARY KEY (TRSNAME)
```

---

## 備註

- Insert 前會先刪除同 TRSNAME 的舊記錄（確保唯一性），見 UserModule 的 `ucSYS_TRSFILES_onBeforeInsert` 鉤子。
- DETAIL2TABLE、DETAIL3TABLE 為後期擴充欄位，支援多層明細的交易定義。
- TRSFORMAT 使用 `nvarchar(max)` 儲存完整的交易規則 JSON，Oracle 版本使用 CLOB 處理大文字。
- 與 SYS_DOCFILES（Word 報表）、SYS_XLSFILES（Excel 報表）同屬 EEP 設計工具的文件體系。
