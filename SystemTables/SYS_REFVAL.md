# SYS_REFVAL

## 用途

**下拉選單參照定義表**（Reference Value Definition / Dropdown Source）。

SYS_REFVAL 定義下拉選單（RefVal）的資料來源，包含查詢語句、顯示欄位、值欄位等。COLDEF 的 `DD_NAME` 欄位關聯此表的 `REFVAL_NO`，實現欄位的下拉選單功能。

### 使用場景

| 場景 | 說明 |
|------|------|
| **下拉選單定義** | COLDEF.DD_NAME → SYS_REFVAL.REFVAL_NO，定義欄位的下拉資料來源 |
| **自訂查詢** | SELECT_COMMAND 可設定自訂 SQL 查詢作為下拉來源（覆蓋 TABLE_NAME 的預設查詢） |
| **多欄顯示** | 搭配 SYS_REFVAL_D1 定義下拉視窗顯示哪些欄位 |
| **前端 RefVal 元件** | `Refval.cs` / `Combobox.cs` / `Autocomplete.cs` / `Submenu.cs` 等編輯器元件讀取此表生成下拉選單 |
| **流程活動** | `Instance.cs` 中 RefRole / RefUser 類型的 SendTo 可能引用 RefVal 定義 |

### 關聯表

```
COLDEF.DD_NAME ──> SYS_REFVAL.REFVAL_NO
SYS_REFVAL.REFVAL_NO ──> SYS_REFVAL_D1.REFVAL_NO（下拉視窗欄位定義，一對多）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPRWDTools.Core/Editors/Refval.cs` | RefVal 編輯器元件：讀取 SYS_REFVAL 定義生成下拉 |
| `EEPRWDTools.Core/Editors/Combobox.cs` | ComboBox 編輯器：也可引用 RefVal 定義 |
| `EEPRWDTools.Core/Editors/Autocomplete.cs` | 自動完成編輯器：引用 RefVal 定義 |
| `EEPRWDTools.Core/Editors/Submenu.cs` | 子選單編輯器：引用 RefVal 定義 |
| `ParserLibrary/ModuleParserExtension.cs` | 模組解析器：解析 RefVal 設定 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **REFVAL_NO** | `varchar(30)` PK | NOT NULL | 參照定義識別碼（如 `CustomerList`） |
| **DESCRIPTION** | `varchar(250)` | NULL | 參照定義描述 |
| **TABLE_NAME** | `varchar(30)` | NULL | 資料來源表名 |
| **CAPTION** | `varchar(30)` | NULL | 下拉視窗標題 |
| **DISPLAY_MEMBER** | `varchar(30)` | NULL | 顯示欄位名稱（下拉時顯示的文字欄位） |
| **VALUE_MEMBER** | `varchar(30)` | NULL | 值欄位名稱（選取後寫入的欄位） |
| **SELECT_ALIAS** | `varchar(250)` | NULL | 查詢別名 |
| **SELECT_COMMAND** | `varchar(250)` | NULL | 自訂 SQL 查詢語句（覆蓋 TABLE_NAME） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (REFVAL_NO)
```

---

## 備註

- SELECT_COMMAND 可設定自訂 SQL，支援複雜查詢（如 JOIN、WHERE 過濾）。
- DISPLAY_MEMBER + VALUE_MEMBER 是下拉選單的核心：顯示 DISPLAY_MEMBER 的文字，選取後寫入 VALUE_MEMBER 的值。
- 若設定了 SELECT_COMMAND，則 TABLE_NAME 會被忽略。
- 無專用 infocommand，由前端元件透過 DataModule 通用框架查詢。
