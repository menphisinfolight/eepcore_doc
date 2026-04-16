# SYS_PARAS

## 用途

**系統參數表**（System Parameters）。

SYS_PARAS 儲存系統層級的參數設定，以 COLUMNNAME + VALUE 的 key-value 方式管理各種系統配置。同一個 COLUMNNAME 可以有多筆 VALUE（如下拉選單的選項列表）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **參數管理（設計端）** | `sysParasColumn` infocommand 以 GROUP BY COLUMNNAME 列出所有參數名稱；`sysParas` 以主從關係顯示每個參數的值列表 |
| **參數儲存** | `SysparasProvider.SaveSysParas()` 儲存參數（由前端 Sysparas.cshtml 呼叫） |
| **參數刪除** | `ucSysParas_onBeforeApply()` 先 DELETE 整個 COLUMNNAME 再 INSERT 新值 |
| **下拉選單資料來源** | 表單設計中，SYS_PARAS 可作為下拉選單的選項來源（COLDEF 的 DD_NAME 指向 COLUMNNAME） |
| **Word 報表下拉** | `WordProvider` 解析報表設定時，將 sysParasArr 寫入 SYS_PARAS 供下拉選單使用 |
| **Excel 報表** | `ExcelProvider` / `ParserHelper` 讀取 SYS_PARAS 參數值 |

### 資料結構特性

SYS_PARAS **沒有主鍵**。同一 COLUMNNAME 可有多筆記錄（每個 VALUE 一筆），形成一對多的參數→值列表：

```
COLUMNNAME='Department', TITLE='部門', VALUE='資訊部'
COLUMNNAME='Department', TITLE='部門', VALUE='業務部'
COLUMNNAME='Department', TITLE='部門', VALUE='人資部'
```

### 關聯表

```
SYS_PARAS ──(COLUMNNAME)──< sysParas infocommand（值列表）
          ──(COLUMNNAME)──  sysParasColumn infocommand（參數名稱列表，GROUP BY）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/SysparasProvider.cs` | `SaveSysParas()`：儲存參數值（繼承自 UserDesignProvider） |
| `SystemTable.Core/UserModule.cs` | `ucSysParas_onBeforeApply()`：DELETE FROM SYS_PARAS WHERE COLUMNNAME = @name（先全刪再全插） |
| `EEPGlobal.Core/Provider/WordProvider.cs` | `SaveSysParas()`：Word 報表匯入時寫入下拉選單參數 |
| `EEPGlobal.Core/Provider/ExcelProvider.cs` | 讀取 SYS_PARAS 參數值 |
| `EEPServerTools.Core/Utility/ParserHelper.cs` | 報表系統讀取 SYS_PARAS |
| `EEPWebClient.Core/design/server/SystemTable.json` | `sysParasColumn`：GROUP BY COLUMNNAME；`sysParas`：WHERE VALUE != ''；`idsSysParas`：主從關係 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **COLUMNNAME** | `nvarchar(50)` | NOT NULL | 參數名稱（如 `Department`、`Position`） |
| **TITLE** | `nvarchar(50)` | NULL | 參數標題/顯示名稱 |
| **VALUE** | `nvarchar(50)` | NULL | 參數值 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 無主鍵

此表無主鍵定義。`sysParas` infocommand 的 keys 設為 `COLUMNNAME,VALUE`（邏輯鍵），但非資料庫主鍵。

---

## 資料生命週期

```
查詢參數列表：設計端 Sysparas.cshtml
      → sysParasColumn infocommand
      → SELECT COLUMNNAME, MIN(TITLE) AS TITLE FROM SYS_PARAS GROUP BY COLUMNNAME

查詢參數值：選擇某個 COLUMNNAME
      → sysParas infocommand（idsSysParas 主從）
      → SELECT * FROM SYS_PARAS WHERE VALUE != '' AND COLUMNNAME = @COLUMNNAME

儲存：前端提交 → SysparasProvider.SaveSysParas()
      → DELETE + INSERT

刪除整個參數：ucSysParas_onBeforeApply()
      → DELETE FROM SYS_PARAS WHERE COLUMNNAME = @name
```

---

## 備註

- 不同於 ASP.NET 的 appsettings.json，EEP 將部分系統參數存在資料庫中，方便動態調整不需重啟。
- `sysParas` infocommand 的查詢條件 `WHERE VALUE != ''` 會過濾掉空值記錄。
- 儲存採「先全刪再全插」策略（DELETE WHERE COLUMNNAME → INSERT 所有新值）。
- VALUE 長度 `nvarchar(50)`，不適合儲存大量文字，只適合簡短的參數值或選項。
- 在表單設計中，SYS_PARAS 可作為下拉選單的資料來源，類似 SYS_REFVAL 但更簡單（無需定義 TABLE_NAME / SELECT_COMMAND）。
