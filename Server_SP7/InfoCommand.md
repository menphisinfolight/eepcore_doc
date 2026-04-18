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
| **runtimeDatabase** | string | 文字 | 指定 Runtime 執行的資料庫。⚠️ **SP7 EEP Core 尚未實作**（只有 EEPCloud 有效），SP8 計畫補上（[#482552](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482552)） |
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

### 範例 8：前端全域變數搭配 onBeforeExecuteSQL（[#482018](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482018)）

**情境**：前端選一筆資料後，用 Button 把條件傳進 InfoCommand 的動態 SQL。

**前端**：
```javascript
$('#btnApply').click(function () {
    var row = $('#dgMain').datagrid('getSelected');
    $.setVariableValue('myFilter', row.ABC);   // 存全域變數（字串）
    $('#dgView').datagrid('reload');           // 重新查詢
});
```

**後端**：
```csharp
public string cmdView_onBeforeExecuteSQL(object sender, string sql, List<string> whereStrs)
{
    var clientInfo = (sender as InfoCommand).Module.ClientInfo;
    var filter = clientInfo.Variables.Get("myFilter");  // 讀全域變數

    if (!string.IsNullOrEmpty(filter))
    {
        // 將原 SQL 包在子查詢中，外層加條件（處理多重 subquery 的情境）
        sql = $"SELECT * FROM ({sql}) q WHERE q.ABC = {MarkValue((object)filter)}";
    }
    return sql;
}
```

> 注意：全域變數 `$.setVariableValue` / `ClientInfo.Variables.Get` **只能存字串**（[#481846](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481846)）。

### 範例 9：軟刪除（不真刪只標記）（[#481033](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481033)）

在 UpdateComponent 攔截刪除，改為 UPDATE 標記欄位；InfoCommand 的 SELECT 自動過濾掉已標記的資料。

**InfoCommand 的 commandText**：
```sql
SELECT * FROM ORDERS WHERE IS_DELETED <> 'Y'
```

**UpdateComponent 搭配**：
```csharp
public bool ucOrder_onBeforeDelete(object sender, dynamic row, List<string> sqls)
{
    var db = (sender as UpdateComponent).Module.DbHelper;
    sqls.Add($"UPDATE ORDERS SET IS_DELETED = 'Y', DELETE_DATE = {db.MarkValue((object)DateTime.Now.ToString("yyyyMMdd"))} "
           + $"WHERE ORDER_NO = {db.MarkValue((object)row.ORDER_NO)}");
    return false;  // ← 阻止系統預設的 DELETE
}
```

### 範例 10：Server 原始碼中直接呼叫 InfoCommand（[#481846](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481846)）

在客製 C# 方法中想用設計好的 InfoCommand 取資料（不重寫 SQL）：

```csharp
public JArray 取得訂單資料(string custId)
{
    var module = new DataModule(ClientInfo, "訂單模組");

    // 方式 1：透過 DataModule.GetDataset（最簡）
    var table = module.GetDataset("cmdOrders", new QueryOptions
    {
        WhereItems = new JArray {
            new JObject {
                { "field", "CUST_ID" }, { "operator", "=" }, { "value", custId }
            }
        },
        PageSize = 0   // 取全部
    });
    return (JArray)table.ToJSON();

    // 方式 2：手動取 SQL + 執行（完全掌控）
    // var cmd = module.GetComponent<InfoCommand>("cmdOrders");
    // var sql = cmd.GetSql();
    // using (var db = module.CreateDatabaseHelper(...))
    //     return (JArray)db.ExecuteDataTable(sql).ToJSON();
}
```

## Stored Procedure 使用（[#479140](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479140) / [#482342](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482342)）

### 設定方式

1. `commandText` 填入 SP 名稱（例如 `sp_GetOrders`）
2. `commandType` 設為 **`StoreProcedure`**
3. `Parameters` 集合中加入各參數：
   - `Name`：參數名稱（**不加 `@` 前綴**）
   - `SqlType`：`Char` / `Nchar` / `Number` / `Datetime`
   - `Value`：預設值（或留空由前端帶入）

### 前端傳參的正確時機

`WhereItems` 屬性設值**不會**送進 SP 參數。必須在 `onQuery` 事件手動 push：

```javascript
function dgOrders_onQuery(param) {
    if (!param.whereItems) param.whereItems = [];
    param.whereItems.push({
        field: 'CUST_ID',   // 對應 Parameters 的 Name
        operator: '=',
        value: $('#dfQuery_custId').textbox('getValue')
    });
}
```

### 限制

| 限制 | 說明 |
|------|------|
| **只能給 DataGrid** | Combobox / Options / RefVal 等不能用 SP 當資料來源（缺查詢欄位傳參機制） |
| **參數名不加 @** | 設計介面 Parameters 欄直接寫 `CUST_ID`，系統自動加 `@` |
| **預覽可能慢** | 設計介面的「查看」按鈕會等 SP 執行，大型 SP 需耐心或暫時改 commandText 測試 |

## 跨資料庫 / Schema 管理

### 1. 切換資料庫連線（`Database` 屬性）

常見情境：正式/測試區切換（[#480409](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480409)）：

1. 在 EEP 後台的資料庫連線管理加一個 `PROD` 連線
2. InfoCommand 的 `Database` 屬性設為 `PROD`
3. **前提**：該資料庫要有 EEP 的系統資料表（COLDEF 等）— **SP7 已放寬**，無 COLDEF 也能預覽（[#482117](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482117)）

### 2. CommandText **不支援 `@變數` 動態 Schema**（[#482342](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482342)）

以下寫法**無效**（`Parameters` 只給 SP 用，CommandText 不會替換）：
```sql
-- ❌ 無效
SELECT * FROM @SCHEMA.TABLE
```

正確做法：**準備多個 InfoCommand**（各指不同 Database），前端 JS 動態切 `remoteName`：

```javascript
function dfMaster_onLoad(row) {
    // 依選取值切 Combobox 的資料來源（進而切到不同 Schema）
    if (row.SCHEMA === 'PROD') {
        $('#dfMaster_品號').combobox('options').remoteName = '出貨單.refItemProd';
    } else {
        $('#dfMaster_品號').combobox('options').remoteName = '出貨單.refItemTest';
    }
    $('#dfMaster_品號').combobox('reload');
}
```

### 3. 撈其他 OWNER 的 Table（Oracle）（[#482345](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482345)）

直接寫 `SELECT * FROM OTHER_OWNER.TABLE` 可能會在 EEP 預覽卡住。

**建議**：新增一個以該 OWNER 身份的資料庫連線，InfoCommand 的 `Database` 指向它，CommandText 就只寫 `SELECT * FROM TABLE`。

### 4. 跨 IP JOIN **不支援**（[#482127](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482127)）

InfoCommand / ServerMethod 都綁一個連線，無法 Join 不同 IP 的兩個 DB。若真需要：

- SQL Server：用 **Linked Server** 後 `SELECT ... FROM [DB1].dbo.T1 JOIN [LinkedServer].DB2.dbo.T2`
- 其他：寫 C# 方法分兩次查，在 memory 合併

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

## 常見陷阱與限制（討論區彙整）

| 陷阱 / 限制 | 說明 | 解法 / 對策 | 來源 |
|------------|------|-------------|------|
| **`runtimeDatabase` SP7 無效** | EEP Core 底層未讀此屬性，只 EEPCloud 實作 | 等 SP8，或改用前端 JS 動態切 `remoteName` | [#482552](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482552) |
| **CommandText 不支援 `@變數`** | `Parameters` 只給 SP 用 | 建多個 InfoCommand + JS 切 `remoteName` | [#482342](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482342) |
| **SP 只支援 DataGrid** | Combobox / Options / RefVal 不能用 SP | 改用 Text 命令，或自行組 WhereItems | [#479140](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479140) / [#482342](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482342) |
| **SP 參數名不能加 `@`** | 系統會自動加，手動加會變 `@@Name` | `Name` 欄填 `CUST_ID` 即可 | [#479140](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479140) |
| **SP 前端傳參要在 onQuery push** | whereItems 屬性設值不會送進 SP | 用 `onQuery(param)` 手動 push 條件 | [#479140](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479140) |
| **跨 IP JOIN 不支援** | 單一連線限制 | SQL Server 用 Linked Server；其他用 C# 分兩次查 | [#482127](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482127) |
| **Oracle `OWNER.TABLE` 預覽卡住** | 跨 OWNER 撈資料可能異常 | 新增該 OWNER 權限的連線，Database 指向它 | [#482345](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482345) |
| **`setWhere` 觸發的查詢可能跳過 `onBeforeExecuteSQL`** | 特定情境下 `DatabaseHelper.IsNumeric` 先報錯 | SP7 已知行為，改用 onQuery 或避開此路徑 | [#482058](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482058) |
| **升 SP7 後 InfoCommand+虛擬欄位+BeforeExecuteSQL 報表破** | 多數是 BeforeExecuteSQL 產生的 SQL 在 SP7 語法變嚴 | 逐個 debug 產生後的 SQL，必要時請原廠協助 | [#482007](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482007) |
| **預覽需要 COLDEF 系統表** | 舊版 InfoCommand 查看按鈕要求該資料庫有 COLDEF | **SP7 已放寬**，無 COLDEF 也能預覽 | [#482117](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482117) |
| **全域變數只能存字串** | `$.setVariableValue` / `ClientInfo.Variables` 都是 string | 數值/物件要自己轉字串再傳 | [#481846](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481846) |
| **Join 出來的 InfoCommand 只能 Update 一個 Table** | UpdateComponent 綁 InfoCommand 時只能對應單一 Table | 用多個 UpdateComponent 分別綁，或 onBeforeUpdate 自組 SQL | [#481226](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481226) |

## 備註

- InfoCommand 不直接執行 SQL，而是由 `DataModule.GetDataset()` 呼叫 `GetSql()` 取得 SQL 後，透過 `DatabaseHelper` 執行。
- `SiteField` 屬性可自動加入站台過濾（`WHERE {siteField} = @Site`），用於多站台部署。
- `RuntimeDatabase` 屬性使用 `[MergeKey("id")]`，表示在設計介面中與 id 合併顯示（但 SP7 未實作功能）。
- `ByParameter` 啟用時使用參數化查詢（`@p_0`、`@p_1`），防止 SQL Injection。
