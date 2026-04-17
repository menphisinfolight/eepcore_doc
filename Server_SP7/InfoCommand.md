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

設計介面上對應「事件」分頁，填入事件方法名稱。Runtime 時由 `Module.InvokeMethod()` 以反射呼叫同名的 C# 方法。

| 事件 | 簽名 | 觸發時機 | 說明 |
|------|------|----------|------|
| **onBeforeExecuteSQL** | `string fn(object sender, string sql, List<string> whereStrs)` | `GetSql()` 組 SQL 最後一步 | 可修改 SQL / WHERE 條件，**必須回傳最終 SQL** |
| **onAfterExecuteSQL** | `void fn(object sender, DataTable table)` | `ExecuteDataTable()` 之後 | 可修改查詢結果（新增欄位、加工資料、補資料） |

### 參數說明

| 參數 | 型別 | 說明 |
|------|------|------|
| `sender` | `object` | 觸發事件的 InfoCommand 實例，需 cast：`(sender as InfoCommand)` |
| `sql` | `string` | 原始 SQL（或 SecStyle 產生的 WHERE 已加入後） |
| `whereStrs` | `List<string>` | 前端 WhereItems 轉成的 WHERE 片段清單（可新增/移除） |
| `table` | `DataTable` | 已執行完的結果資料表（可加欄位、改值） |

### 資料庫特定事件

事件名稱加上資料庫後綴即可針對特定資料庫覆寫，**有後綴版本優先執行**：

| 事件名稱 | 套用資料庫 |
|----------|-----------|
| `onBeforeExecuteSQL` | 預設（SQL Server / MySQL / PostgreSQL） |
| `onBeforeExecuteSQL_Oracle` | Oracle |
| `onBeforeExecuteSQL_Informix` | Informix |
| `onBeforeExecuteSQL_DB2` | DB2 |
| `onBeforeExecuteSQL_PostgreSql` | PostgreSQL |

### 事件方法命名規則

在設計介面填入「事件方法名稱」，不是 InfoCommand 的 id。慣例寫法：

```
{commandId}_onBeforeExecuteSQL
{commandId}_onAfterExecuteSQL
```

例如 InfoCommand id 為 `runtimeMenu`，事件欄位填入 `runtimeMenu_onBeforeExecuteSQL`，C# 類別需有同名方法。

## C# 範例

事件方法寫在對應模組的 `.cs` 檔（如 `design/server/模組名稱/模組名稱.cs`），類別繼承 `DataModule`。

### 範例 1：依登入者過濾（非 HR 只看自己的資料）

```csharp
public string 員工資料_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    var clientInfo = (sender as InfoCommand).Module.ClientInfo;
    string currentUser = clientInfo.User;
    string[] userGroups = clientInfo.Groups;

    // 非 HR 群組只能看自己的資料
    if (!userGroups.Contains("HR"))
    {
        whereStrs.Add($"EmployeeID = '{currentUser}'");
    }
    return sql;
}
```

### 範例 2：依登入者權限動態重組 SQL（選單查詢）

```csharp
public string runtimeMenu_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    var clientInfo = (sender as InfoCommand).Module.ClientInfo;
    var groups = new List<string> { "00" };
    if (clientInfo.Groups != null)
    {
        groups.AddRange(clientInfo.Groups);
    }

    var conditions = new string[] {
        $"MENUID IN (SELECT MENUID FROM USERMENUS WHERE USERID = {MarkValue(clientInfo.User)})",
        $"MENUID IN (SELECT MENUID FROM GROUPMENUS WHERE GROUPID IN ({string.Join(",", groups.Select(g => MarkValue(g)))}))"
    };

    return $"{sql} WHERE ITEMTYPE = {MarkValue(clientInfo.Solution)} "
         + $"AND ({string.Join(" OR ", conditions)}) "
         + $"ORDER BY SEQ_NO, MENUID";
}
```

### 範例 3：使用 DbHelper.InsertWherePart 安全插入條件

```csharp
public string menu_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    var command = sender as InfoCommand;
    var database = command.Module.DbHelper;
    var clientInfo = command.Module.ClientInfo;

    // 如 SQL 尚未含 ITEMTYPE 條件，則插入
    if (sql.IndexOf("ITEMTYPE") < 0)
    {
        return database.InsertWherePart(sql, $"ITEMTYPE={MarkValue(clientInfo.Solution)}");
    }
    return sql;
}
```

### 範例 4：Informix 版本（同功能但欄位需加雙引號）

```csharp
public string menu_onBeforeExecuteSQL_Informix(object sender, string sql, List<string> whereStrs)
{
    var command = sender as InfoCommand;
    var database = command.Module.DbHelper;
    var clientInfo = command.Module.ClientInfo;

    if (sql.IndexOf("ITEMTYPE") < 0)
    {
        return database.InsertWherePart(sql, $"\"ITEMTYPE\"={MarkValue(clientInfo.Solution)}");
    }
    return sql;
}
```

### 範例 5：消耗第一個 whereStrs 並改寫 SQL

```csharp
public string userGroup_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    if (string.IsNullOrWhiteSpace(sql)) return sql;

    if (whereStrs != null && whereStrs.Count > 0)
    {
        // 取第一個條件插入特殊位置，然後從清單移除避免重複
        var newSql = $"{sql} AND {whereStrs[0]} WHERE GROUPID IS NULL ORDER BY USERS.USERID";
        whereStrs.RemoveAt(0);
        return newSql;
    }
    return sql;
}
```

### 範例 6：UNION 組合兩個資料來源

```csharp
public string org_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    if (whereStrs.Count > 0 && whereStrs[0] == "Runtime")
    {
        whereStrs.RemoveAt(0);
        var orgSql = "SELECT 'R' AS TYPE, ORG_NO, ORG_DESC FROM SYS_ORG "
                   + $"WHERE {string.Join(" AND ", whereStrs)}";
        var roleSql = "SELECT 'U' AS TYPE, ROLE_ID AS ORG_NO, GROUPNAME AS ORG_DESC "
                    + $"FROM SYS_ORGROLES WHERE {string.Join(" AND ", whereStrs)}";

        whereStrs.Clear();  // 清空避免框架再次附加
        return $"{orgSql} UNION {roleSql}";
    }
    return sql;
}
```

### 範例 7：OnAfterExecuteSQL 加工結果

```csharp
public void 員工資料_onAfterExecuteSQL(object sender, DataTable table)
{
    // 新增計算欄位
    if (!table.Columns.Contains("FULLNAME"))
    {
        table.Columns.Add("FULLNAME", typeof(string));
    }

    foreach (DataRow row in table.Rows)
    {
        row["FULLNAME"] = $"{row["LASTNAME"]}{row["FIRSTNAME"]}";

        // 遮蔽敏感欄位
        var clientInfo = (sender as InfoCommand).Module.ClientInfo;
        if (!clientInfo.Groups.Contains("HR"))
        {
            row["SALARY"] = "***";
        }
    }
    table.AcceptChanges();
}
```

## 常用方法

事件方法中可透過 `sender as InfoCommand` 取得：

| 存取路徑 | 說明 |
|----------|------|
| `cmd.Module` | 所屬 DataModule |
| `cmd.Module.ClientInfo` | 登入者資訊（User、Groups、Solution、Database） |
| `cmd.Module.DbHelper` | 資料庫 Helper（`ExecuteDataTable`、`InsertWherePart`、`MarkValue` 等） |
| `cmd.CommandText` | 原始 SQL 語句 |
| `cmd.Keys` | 主鍵欄位 |
| `this.MarkValue(value)` | 字串值加引號跳脫（模組繼承的方法） |

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
