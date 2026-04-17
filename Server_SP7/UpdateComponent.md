# UpdateComponent

> `EEPServerTools.Core/Components/UpdateComponent.cs` — 620 行
> 繼承：`UpdateBaseComp` → `ServerComponent` → `Component`

## 用途

**資料更新元件**（Update Component）。

UpdateComponent 負責將前端提交的 inserted/updated/deleted 資料轉換為 INSERT/UPDATE/DELETE SQL 並執行。設計師在模組 JSON 中以 `"type": "updatecomponent"` 宣告，綁定一個 InfoCommand 作為目標資料表。支援欄位屬性控制、預設值、自動編號、XSS 驗證、加密、文字轉換等。

## JSON 設定範例

```json
{
  "type": "updatecomponent",
  "id": "ucMenu",
  "infocommand": "menu",
  "onBeforeInsert": "ucMenu_onBeforeInsert"
}
```

含欄位屬性的進階範例：

```json
{
  "type": "updatecomponent",
  "id": "ucMenuUser",
  "infocommand": "menuUser",
  "fields": [{
    "field": "USERNAME",
    "originalColumn": "",
    "insertEnable": false,
    "defaultValue": "",
    "defaultMode": "Insert",
    "trimLength": ""
  }]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼（唯一） |
| **infocommand** | string | 元件選擇器 `[ControlEditor]` | — | 綁定的 InfoCommand ID（目標資料表） |
| **fields** | FieldAttr[] | 集合編輯器 `[CollectionEditor]` | [] | 欄位屬性設定（見下方） |
| **validateXss** | bool | 勾選框 `[CheckboxEditor]` | true | 是否啟用 XSS 驗證 |
| **rowsAffectCheck** | bool | 勾選框 `[CheckboxEditor]` | false | 是否檢查 SQL 影響筆數（用於樂觀鎖） |
| **updateIfExists** | bool | 勾選 | false | INSERT 時若主鍵已存在，自動改為 UPDATE |
| **textTransform** | TextTransformType | 下拉 | None | 文字轉換：None / Capitalize / Uppercase / Lowercase |
| **isolationLevel** | IsolationLevel | 下拉 | ReadCommitted | 資料庫交易隔離層級：ReadCommitted（預設）/ ReadUncommitted（允許讀取未提交資料） |

## 欄位屬性（FieldAttr）

透過 `fields` 陣列設定每個欄位的特殊行為：

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 欄位名稱 |
| **originalColumn** | string | 文字 | "" | 實際資料庫欄位名（當前端欄位名與 DB 不同時） |
| **insertEnable** | bool | 勾選框 | true | INSERT 時是否包含此欄位 |
| **updateEnable** | bool | 勾選框 | true | UPDATE 時是否包含此欄位 |
| **defaultValue** | string | 預設值編輯器 `[ValueOptionEditor]` | "" | 預設值設定（支援 constant / varaible / function） |
| **defaultMode** | DefaultMode | 下拉 | Both | 預設值套用時機：Insert / Update / Both |
| **trimLength** | int | 數字框 `[NumberboxEditor]` | 0 | 截斷長度（0 = 不截斷） |
| **encryption** | bool | 勾選 | false | 是否加密儲存（需搭配 InfoCommand.encryptionKey） |

### DefaultMode 列舉

| 值 | 說明 |
|----|------|
| `Insert` | 僅在 INSERT 時套用預設值 |
| `Update` | 僅在 UPDATE 時套用預設值 |
| `Both` | INSERT 和 UPDATE 都套用 |

## 事件鉤子

設計介面上對應「事件」分頁，填入事件方法名稱。Runtime 時由 `Module.InvokeMethod()` 以反射呼叫同名的 C# 方法。

| 事件 | 簽名 | 回傳 | 觸發時機 |
|------|------|------|----------|
| **onBeforeApply** | `void fn(object sender, dynamic rows, List<string> sqls)` | — | 整批資料處理前（rows 含 inserted/updated/deleted 三個 JArray） |
| **onBeforeDelete** | `bool fn(object sender, dynamic row, List<string> sqls)` | `false` = 跳過 | 單筆 DELETE 前 |
| **onAfterDelete** | `void fn(object sender, dynamic row, List<string> sqls)` | — | 單筆 DELETE SQL 加入 sqls 後 |
| **onBeforeUpdate** | `bool fn(object sender, dynamic row, List<string> sqls)` | `false` = 跳過 | 單筆 UPDATE 前 |
| **onAfterUpdate** | `void fn(object sender, dynamic row, List<string> sqls)` | — | 單筆 UPDATE SQL 加入 sqls 後 |
| **onBeforeInsert** | `bool fn(object sender, dynamic row, List<string> sqls)` | `false` = 跳過 | 單筆 INSERT 前 |
| **onAfterInsert** | `void fn(object sender, dynamic row, List<string> sqls)` | — | 單筆 INSERT SQL 加入 sqls 後 |
| **onAfterApply** | `void fn(object sender, dynamic rows, List<string> sqls)` | — | 整批 SQL 產生完成後（Transaction 執行前） |
| **onAfterApplied** | `void fn(object sender, dynamic rows, List<string> sqls)` | — | Transaction Commit 之後（由 `DataModule.UpdateDataset` 觸發），適合發通知、寫 log |

### 參數說明

| 參數 | 型別 | 說明 |
|------|------|------|
| `sender` | `object` | 觸發事件的 UpdateComponent 實例，需 cast：`(sender as UpdateComponent)` |
| `row` | `dynamic` | 單筆資料（JObject），可用 `row.COLUMN_NAME` 或 `row["COLUMN_NAME"]` 存取欄位 |
| `rows` | `dynamic` | 整批資料（JObject），含 `rows["inserted"]`、`rows["updated"]`、`rows["deleted"]` 三個 JArray |
| `sqls` | `List<string>` | SQL 清單，可追加自訂 SQL（會一併進入同一 Transaction） |

### Before 事件的回傳值

- `return true`（預設）：繼續執行預設 SQL（INSERT/UPDATE/DELETE）
- `return false`：跳過預設 SQL（通常在事件中自行組 SQL 加入 `sqls`）

### 資料庫特定事件

與 InfoCommand 相同，事件名稱加上資料庫後綴可針對特定資料庫覆寫，**有後綴版本優先執行**：

| 後綴 | 套用資料庫 |
|------|-----------|
| （無） | 預設（SQL Server / MySQL / PostgreSQL） |
| `_Oracle` | Oracle |
| `_Informix` | Informix |
| `_DB2` | DB2 |

### 事件方法命名規則

在設計介面填入「事件方法名稱」。慣例是以 UpdateComponent 的 id 為前綴：

```
{updateComponentId}_onBeforeInsert
{updateComponentId}_onBeforeUpdate
{updateComponentId}_onAfterApplied
```

例如 UpdateComponent id 為 `ucMenu`，事件欄位填入 `ucMenu_onBeforeInsert`。

## C# 範例

事件方法寫在對應模組的 `.cs` 檔（如 `design/server/模組名稱/模組名稱.cs`），類別繼承 `DataModule`。

### 範例 1：官方範本（條件改值）

```csharp
public bool uc你的名稱_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    if (row["你的欄位"] == 你的條件)
    {
        row["你的欄位"] = "你的條件";
    }
    return true;
}
```

### 範例 2：INSERT 前加密密碼 + 寫入建立日期

```csharp
public bool ucUser_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    if (!string.IsNullOrEmpty((string)row.PWD))
    {
        var ePassword = new EncryptHelper().EncryptPassword((string)row.USERID, (string)row.PWD);
        row.PWD = ePassword;
        row.PWD2 = ePassword;
    }
    row.CREATEDATE = DateTime.Today.ToString("yyyyMMdd");
    return true;
}
```

### 範例 3：UPDATE 前依條件決定是否執行（`return false` 用法）

```csharp
public bool ucUser_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    var importMode = (sender as UpdateComponent).Options.ImportMode;
    if (importMode == "insert")
    {
        return false;  // 跳過這筆 UPDATE（不產生 SQL）
    }

    // 密碼欄位有異動才重新加密
    if (row.PWD != row.PWD2)
    {
        var ePassword = !string.IsNullOrEmpty((string)row.PWD)
            ? new EncryptHelper().EncryptPassword((string)row.USERID, (string)row.PWD)
            : string.Empty;
        row.PWD = ePassword;
        row.PWD2 = ePassword;
    }
    return true;
}
```

### 範例 4：UPDATE 前欄位格式正規化（日期/時間去分隔符）

```csharp
public bool ucAgent_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    row.START_DATE = ((string)row.START_DATE).Replace("/", "").Replace("-", "");
    row.END_DATE   = ((string)row.END_DATE).Replace("/", "").Replace("-", "");
    row.START_TIME = ((string)row.START_TIME).Replace(":", "");
    row.END_TIME   = ((string)row.END_TIME).Replace(":", "");

    // 時間補 00 秒（HHmm → HHmmss）
    if (row.START_TIME != null && row.START_TIME.ToString().Length == 4)
        row.START_TIME = row.START_TIME.ToString() + "00";
    if (row.END_TIME != null && row.END_TIME.ToString().Length == 4)
        row.END_TIME = row.END_TIME.ToString() + "00";

    return true;
}
```

### 範例 5：INSERT 前自動編號 + 追加關聯表 SQL

```csharp
public bool ucMenu_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    var database = (sender as UpdateComponent).Module.DbHelper;
    var clientInfo = (sender as UpdateComponent).Module.ClientInfo;

    // 自動編號：取目前最大 MENUID + 1
    var maxID = GetMaxID(database.ExecuteDataTable("SELECT MENUID FROM MENUTABLE"), "MENUID");
    row.ITEMTYPE = clientInfo.Solution;
    row.MENUID = (maxID + 1).ToString();

    if (row.CAPTION == null) row.CAPTION = string.Empty;

    // 追加 SQL：自動為新選單授權給「所有人」群組
    sqls.Add($"DELETE FROM GROUPMENUS WHERE MENUID = {MarkValue(row.MENUID)}");
    sqls.Add($"INSERT INTO GROUPMENUS (GROUPID, MENUID) VALUES ({MarkValue("00")},{MarkValue(row.MENUID)})");

    return true;
}
```

### 範例 6：onBeforeApply（整批前處理，處理匯入模式）

```csharp
public void ucUser_onBeforeApply(object sender, dynamic rows, List<string> sqls)
{
    var importMode = (sender as UpdateComponent).Options.ImportMode;
    if (string.IsNullOrEmpty(importMode)) return;

    if (importMode == "replace")
    {
        // replace 模式：先清空整個資料表
        var database = (sender as UpdateComponent).Module.DbHelper;
        sqls.Add(database.GetDeleteAllSql("USERS"));
    }
    else
    {
        // 其他匯入模式：開啟 UpdateIfExists（重複主鍵自動改 UPDATE）
        (sender as UpdateComponent).UpdateIfExists = true;
    }
}
```

### 範例 7：onAfterInsert（INSERT 之後追加關聯資料）

```csharp
public void ucGroup_onAfterInsert(object sender, dynamic row, List<string> sqls)
{
    var importMode = (sender as UpdateComponent).Options.ImportMode;
    if (string.IsNullOrEmpty(importMode)) return;

    // 先清空該群組既有的使用者關聯
    sqls.Add($"DELETE FROM USERGROUPS WHERE GROUPID = {MarkValue(row.GROUPID)}");

    // 再依 row.Users（匯入檔帶來的清單）逐筆插入
    foreach (var user in row.Users)
    {
        sqls.Add($"INSERT INTO USERGROUPS (USERID, GROUPID) VALUES ({MarkValue(user)},{MarkValue(row.GROUPID)})");
    }
}
```

### 範例 8：Informix 版本（欄位/表名需加雙引號）

```csharp
public void ucGroup_onAfterInsert_Informix(object sender, dynamic row, List<string> sqls)
{
    var importMode = (sender as UpdateComponent).Options.ImportMode;
    if (string.IsNullOrEmpty(importMode)) return;

    sqls.Add($"DELETE FROM \"USERGROUPS\" WHERE \"GROUPID\" = {MarkValue(row.GROUPID)}");
    foreach (var user in row.Users)
    {
        sqls.Add($"INSERT INTO \"USERGROUPS\" (\"USERID\", \"GROUPID\") VALUES ({MarkValue(user)},{MarkValue(row.GROUPID)})");
    }
}
```

### 範例 9：onAfterApplied（Transaction Commit 之後）

```csharp
public void uc訂單_onAfterApplied(object sender, dynamic rows, List<string> sqls)
{
    // 此時資料已成功寫入，可發通知或做外部呼叫
    var clientInfo = (sender as UpdateComponent).Module.ClientInfo;

    foreach (JObject inserted in rows["inserted"] as JArray)
    {
        // 寄送通知信（資料已入庫，失敗不會影響 Transaction）
        // Mail.Send(...);
    }
}
```

## 常用方法

事件方法中透過 `sender as UpdateComponent` 取得：

| 存取路徑 | 說明 |
|----------|------|
| `uc.Module` | 所屬 DataModule |
| `uc.Module.ClientInfo` | 登入者資訊（User、Groups、Solution、Database） |
| `uc.Module.DbHelper` | 資料庫 Helper（`ExecuteDataTable`、`GetDeleteAllSql`、`MarkValue` 等） |
| `uc.Options.ImportMode` | 匯入模式字串（`"replace"` / `"insert"` / `"update"` / 空 = 一般前端提交） |
| `uc.UpdateIfExists` | 可動態設為 true 啟用「INSERT 若主鍵存在則改 UPDATE」 |
| `uc.Infocommand` | 綁定的 InfoCommand id |
| `this.MarkValue(value)` | 字串值加引號跳脫（模組繼承的方法，防 SQL Injection） |

## GetSqls() 流程

```
GetSqls(rows)
│
├─ 1. OnBeforeApply(rows, sqls) — 整批前處理
│
├─ 2. deleted → GetDeleteSqls()
│     ├─ OnBeforeDelete(row) → false 則跳過
│     ├─ 刪除明細（透過 InfoDataSource 找關聯表）
│     ├─ DELETE FROM {table} WHERE {keys}
│     └─ OnAfterDelete(row)
│
├─ 3. updated → GetUpdateSqls()
│     ├─ TextTransform 文字轉換
│     ├─ OnBeforeUpdate(row) → false 則跳過
│     ├─ UPDATE {table} SET {columns} WHERE {keys}
│     │   ├─ 跳過 ReadOnly 欄位
│     │   ├─ FieldAttr.updateEnable 控制
│     │   ├─ FieldAttr.defaultValue 套用（DefaultMode != Insert）
│     │   ├─ FieldAttr.trimLength 截斷
│     │   ├─ FieldAttr.encryption 加密
│     │   └─ ValidateXss 驗證
│     └─ OnAfterUpdate(row)
│
├─ 4. inserted → GetInsertSqls()
│     ├─ TextTransform 文字轉換
│     ├─ AutoNumber 自動編號（Init + GetValue）
│     ├─ 偵測 Identity 欄位
│     ├─ DuplicateCheck → 重複則拋錯
│     ├─ UpdateIfExists → 重複則改為 UPDATE
│     ├─ OnBeforeInsert(row) → false 則跳過
│     ├─ ParentValues 注入（明細表從主表繼承值）
│     ├─ INSERT INTO {table} ({columns}) VALUES ({values})
│     │   ├─ 跳過 ReadOnly 欄位
│     │   ├─ FieldAttr.insertEnable 控制
│     │   ├─ FieldAttr.defaultValue 套用（DefaultMode != Update）
│     │   ├─ FieldAttr.encryption 加密
│     │   └─ ValidateXss 驗證
│     └─ OnAfterInsert(row)
│
├─ 5. AutoNumber.GetSqls() — 回寫流水號到 SYSAUTONUM
│
├─ 6. InfoTransaction.GetSqls() — 交易回寫
│
└─ 7. OnAfterApply(rows, sqls) — 整批後處理
```

## XSS 驗證

當 `validateXss = true` 時，所有字串值都會通過正則驗證：

```
允許字元：中文、英數、常見符號（`~!@#$%^&*()-+=?:"{}|,./;'\[]）
不允許：HTML 標籤（<script> 等）
```

驗證失敗會拋出 `validateXss:{value}` 例外。

## 明細表刪除連動

DELETE 主表時，自動透過 InfoDataSource 找到所有關聯的明細表，先刪除明細再刪除主表：

```
DELETE 主表 row
  → 找所有 InfoDataSource（MasterCommand == this.Infocommand）
  → 對每個明細表產生 DELETE SQL
  → 最後 DELETE 主表
```

## 備註

- UpdateComponent 不直接執行 SQL，而是回傳 `List<string>` 給 `DataModule.UpdateDataset()` 批次執行。
- 整個流程在同一個 Transaction 中（Commit 或 Rollback）。
- `OnAfterApplied` 是在 Transaction Commit **之後**觸發，適合做非資料庫操作（如發通知）。
- `_insertedRow` 記錄最新插入的主表行，供 InfoTransaction 和 InfoDataSource 使用。
- Identity 欄位會被自動偵測（`DbHelper.IsIdentity`），INSERT 後可透過 `GetInsertedValue` 取回自動產生的 ID。
