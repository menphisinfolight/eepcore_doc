# Enum.cs

- **檔案路徑**: `EEPServerTools.Core/Enum.cs`
- **行數**: 68 行
- **命名空間**: `EEPServerTools.Core`

## 用途

定義 EEPServerTools.Core 各元件共用的列舉型別（Enum）。這些列舉控制了指令類型、權限樣式、SQL 參數型別、交易模式、更新模式等核心行為設定，供多個 Server 元件引用。

## 列舉定義

### CommandType

SQL 指令的類型。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `Text` | 純文字 SQL 指令 |
| 1 | `StoreProcedure` | 預存程序（Stored Procedure） |

**使用元件**: [[InfoCommand]]、[[DataModule]]

---

### SecStyle

資料安全過濾樣式，決定 SQL 指令附加的權限過濾條件。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `None` | 不加任何權限過濾 |
| 1 | `User` | 依使用者過濾 |
| 2 | `Group` | 依群組過濾 |
| 3 | `FlowHistory` | 依簽核歷程過濾 |
| 4 | `Role` | 依角色過濾 |
| 5 | `Org` | 依組織過濾 |

**使用元件**: [[InfoCommand]]

---

### SqlType

SQL 參數的資料型別，用於指令參數繫結時的型別對應。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `Char` | 字元型別（varchar） |
| 1 | `Nchar` | Unicode 字元型別（nvarchar） |
| 2 | `Number` | 數值型別 |
| 3 | `Datetime` | 日期時間型別 |

**使用元件**: [[InfoCommand]]、[[DataModule]]

---

### TransMode

交易明細的新增模式，控制 InfoTransaction 處理明細資料時的行為。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `AutoAppend` | 自動新增：依條件判斷是否新增明細 |
| 1 | `SyncAppend` | 同步新增：與主檔同步新增明細 |
| 2 | `AlwaysAppend` | 強制新增：每次都新增明細 |
| 3 | `Ignore` | 忽略：不處理明細 |
| 4 | `Exception` | 例外：遇到時拋出例外 |

**使用元件**: [[InfoTransaction]]

---

### UpdateMode

欄位更新模式，決定資料異動時欄位值的處理方式。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `Replace` | 取代：直接以新值覆蓋舊值 |
| 1 | `Increase` | 遞增：在原值基礎上加上新值 |
| 2 | `Decrease` | 遞減：在原值基礎上減去新值 |
| 3 | `DecreaseNotZero` | 遞減不為零：遞減但結果不可為負數 |
| 4 | `ReplaceNegative` | 取代允許負數：取代且允許負值 |
| 5 | `WriteBack` | 回寫：將值回寫至來源 |
| 6 | `WriteBackInc` | 回寫遞增：回寫時以遞增方式處理 |
| 7 | `WriteBackDec` | 回寫遞減：回寫時以遞減方式處理 |

**使用元件**: [[InfoTransaction]]

---

### TransScope

交易範圍，控制交易處理的涵蓋範圍。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `All` | 全部：處理所有項目 |
| 1 | `One` | 單筆：僅處理單一項目 |

**使用元件**: [[ServerMove]]

---

### IsolationLevel

交易隔離等級，控制資料庫交易的讀取隔離行為。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `ReadCommitted` | 已認可讀取：只能讀取已提交的資料（預設） |
| 1 | `ReadUncommitted` | 未認可讀取：可讀取未提交的資料（髒讀） |

**使用元件**: [[UpdateComponent]]

---

### DefaultMode

預設值套用模式，決定欄位預設值在何種操作時生效。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `Insert` | 新增時套用預設值 |
| 1 | `Update` | 修改時套用預設值 |
| 2 | `Both` | 新增與修改時皆套用預設值 |

**使用元件**: [[UpdateComponent]]

---

### LineType

線上人員類型，區分線上查詢的對象類別。

| 值 | 名稱 | 說明 |
|---|------|------|
| 0 | `User` | 使用者 |
| 1 | `Group` | 群組 |

**使用元件**: [[InfoLine]]

---

## 備註

- 所有列舉皆定義在 `EEPServerTools.Core` 命名空間下，為公開（public）型別。
- 列舉值皆從 0 開始，依序遞增（未指定自訂數值）。
- `CommandType` 和 `SqlType` 被多個元件共用，是最基礎的列舉型別。
- `UpdateMode` 提供豐富的欄位更新策略，特別是 `WriteBack` 系列用於跨檔回寫場景（如庫存扣減後回寫來源單據）。
- `SecStyle` 的多種過濾樣式對應 EEP 系統的權限架構（使用者、群組、角色、組織、簽核歷程）。
