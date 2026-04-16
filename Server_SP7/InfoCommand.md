# InfoCommand

> `EEPServerTools.Core/Components/InfoCommand.cs` — 481 行
> 繼承：`SelectComp` → `ServerComponent` → `Component`

## 用途

**資料查詢命令元件**（Info Command）。

InfoCommand 是 EEP Core 最核心的伺服器元件，負責定義一個 SQL 查詢命令。設計師在模組 JSON 中以 `"type": "infocommand"` 宣告，指定 SQL 語句、主鍵、安全過濾、分頁、快取等設定。Runtime 時由 `DataModule.GetDataset()` 呼叫 `InfoCommand.GetSql()` 產生最終 SQL。

## JSON 設定範例

```json
{
  "type": "infocommand",
  "id": "menu",
  "commandText": "SELECT * FROM MENUTABLE ORDER BY SEQ_NO, MENUID",
  "keys": "MENUID",
  "onBeforeExecuteSQL": "runtimeMenu_onBeforeExecuteSQL"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **id** | string | 文字 | 元件識別碼（唯一） |
| **commandText** | string | SQL 編輯器 `[SqlEditor]` | SQL 查詢語句（SELECT）或預存程序名稱 |
| **commandType** | CommandType | 下拉 | `Text`（SQL）或 `StoreProcedure`（預存程序） |
| **keys** | string | 欄位選擇器 `[ColumnEditor]` | 主鍵欄位（逗號分隔，如 `"MENUID,USERID"`） |
| **database** | string | 文字 | 指定資料庫（空 = 使用 ClientInfo.Database） |
| **commandTimeout** | int | 數字框 `[NumberboxEditor]` | 查詢逾時秒數（預設 30） |
| **selectPaging** | bool | 勾選 | 是否啟用伺服器端分頁 |
| **descent** | bool | 勾選 | 是否依主鍵倒序排列 |
| **trimStringData** | bool | 勾選 | 是否去除字串尾端空白 |
| **nonLogon** | bool | 勾選 | 是否允許未登入存取 |

## 安全過濾（SecStyle）

控制資料列級別的存取權限。設計師設定 `secStyle` + `secField`，Runtime 時自動在 SQL 加入 WHERE 條件。

| SecStyle | secField 意義 | WHERE 條件 |
|----------|--------------|------------|
| `User` | 使用者欄位 | `WHERE {secField} = @User` |
| `Group` | 群組欄位 | `WHERE {secField} IN (@Groups)` |
| `Role` | 角色欄位 | `WHERE {secField} IN (@Roles)` |
| `Org` | 組織角色欄位 | `WHERE {secField} IN (@OrgRoles)` |
| `FlowHistory` | 流程旗標欄位 | `WHERE FlowFlag 的 InstanceID IN (FlowHistory ∪ FlowToDo)` |
| `None` | — | 不過濾 |

### SecMultiple

當 `secMultiple = true` 時，secField 欄位儲存多值（逗號分隔），過濾改用 `LIKE '%,value,%'` 而非 `=`。

### SecExcept

設定排除名單，符合的使用者/群組不受安全過濾限制。

### SecColumns — 欄位級別安全

```json
"secColumns": [
  { "field": "SALARY", "users": "admin", "groups": "HR", "encryption": false }
]
```

指定欄位只有特定使用者/群組可見，其他人該欄位會被遮蔽。`encryption = true` 時以加密方式遮蔽。

## 快取機制

| 屬性 | 說明 |
|------|------|
| **cacheData** | 是否啟用前端快取 |
| **cacheTTL** | 快取有效分鐘數（預設 1440 = 24 小時） |
| **cacheSynchronize** | 同步模式：`EveryTime`（每次重查）、`Smart`（只查新資料）、`None`（不同步） |
| **cacheVerifyField** | Smart 模式的驗證欄位（用於判斷是否有新資料） |

### Smart 快取流程

```
前端帶 caches 參數 → DataModule.GetDataset()
  → 找到 cacheVerifyField 的最新值
  → SELECT ... WHERE {verifyField} > @latestValue
  → 有新資料 → 回傳完整資料
  → 無新資料 → 回傳 null（前端續用快取）
```

## 事件鉤子

| 事件 | 簽名 | 說明 |
|------|------|------|
| **onBeforeExecuteSQL** | `string fn(sender, sql, whereStrs)` | SQL 執行前，可修改 SQL 和 WHERE 條件，回傳修改後的 SQL |
| **onAfterExecuteSQL** | `void fn(sender, dataTable)` | SQL 執行後，可修改結果 DataTable |

### 資料庫特定事件

事件名稱加上資料庫後綴即可針對特定資料庫覆寫：
- `onBeforeExecuteSQL` — 通用（SQL Server / MySQL）
- `onBeforeExecuteSQL_Oracle` — Oracle 專用
- `onBeforeExecuteSQL_Informix` — Informix 專用

## GetSql() 流程

```
1. SetSecColumns() → 設定欄位級別安全（遮蔽/加密）
2. SetSecStrs() → 根據 SecStyle 加入 WHERE 條件
3. 解析 WhereItems（前端傳入的查詢條件）
   → 轉換 operator：= / like / in / %（前置模糊）/ %%（全模糊）
   → 日期欄位自動轉換 % → =
   → ByParameter 模式：使用參數化查詢
   → or 標記：收集到 orStrs，最後以 OR 合併
4. 處理 Descent 排序
5. 觸發 OnBeforeExecuteSQL 事件（可修改最終 SQL）
6. 回傳 SQL 字串
```

## WhereItems 運算子

前端傳入的查詢條件格式：

```json
{ "field": "USERNAME", "operator": "%%", "value": "張" }
```

| operator | 轉換結果 | 說明 |
|----------|---------|------|
| `=`（或空） | `field = 'value'` | 精確比對 |
| `%` | `field LIKE 'value%'` | 前置模糊（日期欄位自動改為 `=`） |
| `%%` | `field LIKE '%value%'` | 全模糊 |
| `in` | `field IN ('v1','v2')` | 多值比對 |
| `or: true` | 以 OR 合併 | 與其他條件用 OR 而非 AND |

## 內部類別

### Parameter

預存程序的參數定義：

| 屬性 | 說明 |
|------|------|
| Name | 參數名稱 |
| SqlType | Char / Nchar / Number / Datetime |
| Value | 參數值 |

### SecColumn

欄位安全定義：

| 屬性 | 說明 |
|------|------|
| Field | 欄位名稱 |
| Users | 可見的使用者（逗號分隔） |
| Groups | 可見的群組（逗號分隔） |
| Encryption | 是否加密遮蔽 |

## 其他方法

| 方法 | 說明 |
|------|------|
| `GetSchemaTable()` | 取得 SQL 的 Schema 資訊（快取，只查一次） |
| `GetTableName()` | 從 CommandText 解析表名 |
| `ExecuteDataTable()` | 直接執行 SQL（不經 DataModule.GetDataset） |
| `GetOldRow(row)` | 以主鍵查詢更新前的舊資料（用於交易回寫比較） |
| `GetParameters()` | 預存程序參數轉換（WhereItems → Parameter） |

## 備註

- InfoCommand 不直接執行 SQL，而是由 `DataModule.GetDataset()` 呼叫 `GetSql()` 取得 SQL 後，透過 `DatabaseHelper` 執行。
- `SiteField` 屬性可自動加入站台過濾（`WHERE {siteField} = @Site`），用於多站台部署。
- `RuntimeDatabase` 屬性使用 `[MergeKey("id")]`，表示在設計介面中與 id 合併顯示。
- `ByParameter` 啟用時使用參數化查詢（`@p_0`、`@p_1`），防止 SQL Injection。
