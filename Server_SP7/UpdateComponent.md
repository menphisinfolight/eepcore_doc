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

### 範例 10：事件中取得更新前的舊資料（`GetOldRow`）（[#482517](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482517)）

**情境**：想在 `onBeforeUpdate` 比對「修改前 vs 修改後」的差異。

```csharp
public bool uc員工資料表_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    // 正確寫法：從 UpdateComponent 的 Command 呼叫 GetOldRow
    var uc = sender as UpdateComponent;
    var oldRow = uc.GetCommand().GetOldRow(row);

    // 或從 Module 取 UpdateComponent 實例
    // var oldRow = this.GetComponent<UpdateComponent>("uc員工資料表").GetCommand().GetOldRow(row);

    if (oldRow == null) return true;  // 找不到舊資料（罕見）

    // 比對欄位變化
    if (oldRow["薪資"]?.ToString() != row.薪資.ToString())
    {
        // 薪資有變動 → 寫入調薪紀錄
        sqls.Add($"INSERT INTO SALARY_HISTORY (USER_ID, OLD_SALARY, NEW_SALARY, CHANGE_DATE) VALUES "
            + $"({MarkValue((object)row.USER_ID)}, {MarkValue(oldRow["薪資"])}, {MarkValue((object)row.薪資)}, "
            + $"{MarkValue((object)DateTime.Now.ToString("yyyyMMdd"))})");
    }
    return true;
}
```

> 常見錯誤：把 `GetComponent<UpdateComponent>("...")` 的參數寫成 InfoCommand 名稱或 class 名。參數應該是 **UpdateComponent 的 id**。

### 範例 11：DETAIL 的 onBeforeApply 取得 MASTER 初始值（[#474147](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474147)）

**情境**：明細表要知道主檔「修改前的原始值」來比對差異。

**做法**：在主檔 DataForm 的 `onLoad` 把 row 存全域變數，明細事件從變數讀取。

**前端**：
```javascript
function dfMaster_onLoad(row) {
    if ($('#dfMaster').form('status') === 'updated') {
        // 把主檔原始值存進全域變數（只能存字串）
        $.setVariableValue('masterOriginal', JSON.stringify(row));
    }
}
```

**後端明細事件**：
```csharp
public void ucDetail_onBeforeApply(object sender, dynamic rows, List<string> sqls)
{
    var clientInfo = (sender as UpdateComponent).Module.ClientInfo;
    var masterOriginalJson = clientInfo.Variables.Get("masterOriginal");
    if (string.IsNullOrEmpty(masterOriginalJson)) return;

    var masterOriginal = JObject.Parse(masterOriginalJson);
    // 用 masterOriginal 比對...
}
```

> **注意**：全域變數按**裝置 + 瀏覽器快取**隔離，不跨使用者、不跨裝置。

## 事件執行順序的重要觀念

> 來源：[#473877](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473877) / [#471374](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=471374) / [#481331](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481331) / [#472657](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=472657)

常見誤會：「`onAfterInsert` 應該在 INSERT 真正執行**之後**才呼叫」— 但 SQL Profiler 看起來**順序不對**。

### 實際機制

1. 事件方法（`onBeforeXxx` / `onAfterXxx`）的角色是**加 SQL 到 `sqls` 清單**
2. `UpdateComponent.GetSqls()` 回傳整批 SQL
3. `DataModule.UpdateDataset()` 把清單交給 `DbHelper.ExecuteNonQuery(sqls, options)` **一次性**執行（同一個 Transaction）
4. 所以 `onAfterInsert` 的程式碼「跑完」時，INSERT **還沒真正進 DB** — 它只是把 SQL 加進佇列

### 主明細順序

在主明細（Master-Detail）架構下，SQL 執行順序大致是：

```
1. Master Delete SQL（如有）
2. Master Update SQL（如有）
3. Master Insert SQL（含 onAfterInsert 加的 SQL）
4. Detail Delete SQL
5. Detail Update SQL（含明細 onAfterUpdate 加的 SQL）
6. Detail Insert SQL
7. InfoTransaction（TRS）產生的 SQL
```

**結果**：如果在 Detail 的 `onAfterUpdate` 寫「依明細狀態回寫主檔」，此時 Master SQL 早就跑完了 → 回寫邏輯會檢查到**未更新前**的明細。

### 解法

要在主明細資料都異動完成後才處理時，在**主檔 DataForm 的 `onApplied`** 觸發 ServerMethod 回寫：

```javascript
function dfMaster_onApplied(data) {
    // 主明細都已 commit，此時呼叫 serverMethod 回寫
    $.callMethod('模組', '回寫主檔', [data.USER_ID]);
}
```

### 觸發規則

| 事件 | 主明細下的觸發時機 |
|------|-------------------|
| **主檔 UpdateComponent `onAfterApplied`** | ✅ 會觸發（Transaction Commit 之後） |
| **明細 UpdateComponent `onAfterApplied`** | ❌ **不會觸發**（只主檔的會） |
| **主/明細 `onBeforeApply` / `onAfterApply`** | ✅ 各自觸發 |

## InfoTransaction（TRS）搭配

> 來源：[#482459](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482459) / [#482386](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482386) / [#469257](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469257) / [#472634](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=472634)

跨 Table / 跨 DB 的連動寫入，**優先用 TRS 而非自寫 ServerMethod**。

### TRS 的 5 種 TransMode

| TransMode | 說明 |
|-----------|------|
| **AutoAppend** | 以 `TransKeyFields` 找目標表，**找到則 Update、找不到則 Insert** |
| **SyncAppend** | 類似 AutoAppend，但資料異動與原表格同步（新增、修改、**刪除**） |
| **WhenDelete** | 原表刪除時才觸發（配 SyncAppend） |
| **WhenInsert** | 原表新增時才觸發 |
| **WhenUpdate** | 原表修改時才觸發 |

### 典型情境

**需求**：刪除 A 表某筆資料，自動刪除 B 表中對應 KEY 值的資料

**TRS 設定**：
```
TargetTable:     B表
TransMode:       SyncAppend + WhenDelete
TransKeyFields:  KEY
AppendMode:      Delete
```

### 限制與技巧

- **USERS 等系統表在 TargetTable 下拉選不到** — 需**手動輸入表名**（[#469257](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469257)）
- **跨 DB INSERT** — 設定 TargetTable 時指定不同的 Database 連線（[#472634](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=472634)）
- TRS 的設計介面比 ServerMethod 更直覺，但屬性較複雜

## DataPanel 呼叫 UpdateComponent 的條件

> 來源：[#482410](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482410)

**要求**：DataPanel 必須與 DataForm **相同 RemoteName**。

若要把不同表的欄位一起編輯：
1. 在 InfoCommand 用 **LEFT JOIN** 把外部表欄位併進來
2. 在 UpdateComponent 把外部表欄位的 `insertEnable` / `updateEnable` **設為 false**（僅顯示不寫入）
3. 這樣 DataPanel 顯示所有欄位，但 Update 只影響主表

## XSS 驗證進階用法

> 來源：[#473029](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473029) / [#473697](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473697) / [#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032)

### 驗證位置

前端實作：`EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.validate.js`

### 多處設定要同步

若一筆資料會同時經過 DataGrid 顯示與 UpdateComponent 寫入，**兩邊都要關**才能完整支援越南文 / 泰文 / 特殊字元：

```json
// DataGrid
{ "type": "datagrid", "validateXss": false }

// UpdateComponent
{ "type": "updatecomponent", "validateXss": false }
```

### 全域覆寫（系統頁面用）

若要修改系統頁面（如 `Sysusers.cshtml`）的 XSS 行為，在**主頁原始碼**（EEP 的主頁原始碼覆寫機制）注入：

```javascript
// 完全關閉 XSS 驗證（直接 pass through 原值）
$.validateScript = function (value) {
    return value;
};
```

## Flow 相關陷阱

### FlowFlag 欄位存檔後為空（[#473801](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473801)）

**症狀**：自建表單新增存檔，`FLOWINSTANCE` / `FLOWHISTORY` 的 InstanceID 正常，但主表 `FlowFlag` 欄位**空白**。

**原因**：流程活動的 **`Title` 屬性沒填**。

**解法**：確認 Flow 活動的 Title 屬性已設定值。

### onAfterInsert 內 UPDATE 欄位，Flow 讀不到（[#482130](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482130)）

**症狀**：表單有 AutoNumber 自動編號欄位，在 `onAfterInsert` 用 UPDATE 組合「單號+版次」寫進另一欄位（FLOWNO），Flow 判斷用不到這個值。

**原因**：事件產生的 UPDATE 和 Flow 抓資料可能**不在同一個 Transaction**。

**官方建議**：不要開新欄位紀錄組合值，改用 **InfoCommand 虛擬欄位**（用 SQL 拼接即時計算）。若 FLOWNO 需要當 Key，直接讓「單號 + 版次」雙 Key 也能達成。

### 審核中單據刪除控制（[#481945](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481945)）

**要件**：
1. 表單掛載在**選單**上（Menu）
2. 設定**流程參數**
3. Mode 為 **預備**

若複製 InfoCommand / UpdateComponent 做另一個 RWD（常用於不同 SecField），審核中刪除的自動控制**需要重複上述設定**。

## Identity 自增值取得時機

> 來源：[#470099](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=470099)

INT IDENTITY 自增值**不能在 INSERT SQL 中取得**。正確做法是在主檔 `onApplied` reload datagrid，系統會把資料庫產生的 ID 帶回：

```javascript
function dfMaster_onApplied(data) {
    // 系統已自動 reload 取回 Identity，此處 data 含最新 ID
    console.log(data.inserted[0].ID);  // Identity 值
}
```

## 例外訊息顯示 InnerException

> 來源：[#469106](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469106)

**問題**：`throw new Exception("訊息")` 時，前端顯示的是外層 Exception 而非 InnerException。

**解法**（需改 `DataProvider.cs`，**非公版**）：

```csharp
// 在 catch 區塊中
catch (Exception ex) {
    var msg = ex.InnerException?.Message ?? ex.Message;
    throw new Exception(msg);
}
```

## SecColumns Encryption UI 限制

> 來源：[#481353](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481353)

**症狀**：官方文件提到 SecColumns 可設 `Encryption`，但 UpdateComponent / InfoCommand 的 Collection Editor 實際**找不到此勾選框**（SP6/SP7 UI 未暴露）。

**暫時解法**：直接編輯模組 JSON 加入 `"encryption": true`：

```json
"secColumns": [
  { "field": "SALARY", "users": "admin", "groups": "HR", "encryption": true }
]
```

## `throw` 中止整個 Transaction

事件中用 `throw` 可以**中止整批存檔**（無論主檔還是明細，無論哪個事件時機點）。

### 規則

| 項目 | 說明 |
|------|------|
| **Rollback 範圍** | 整個 Transaction 的所有 SQL 都會 Rollback（主表、明細、TRS、AutoNumber 全部退回） |
| **throw 類別** | 用 `throw new EEPException("訊息")`，前端會友善顯示訊息 |
| **用 `throw new Exception`** | 前端會當成「系統錯誤」顯示，訊息可能被框架吃掉或顯示外層錯誤（見 [#469106](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469106)） |
| **`onAfterApplied` 是否觸發** | ❌ 不會（Commit 失敗，Commit 之後的 hook 都不跑） |
| **`onBeforeApply` 無 `return false`** | 它是 `void` 方法，要中止**只能 `throw`** |

### 範例：業務檢核失敗中止存檔

```csharp
public void uc訂單_onBeforeApply(object sender, dynamic rows, List<string> sqls)
{
    var inserted = rows["inserted"] as JArray;
    if (inserted.Count == 0) return;

    foreach (JObject row in inserted)
    {
        // 檢查庫存
        var db = (sender as UpdateComponent).Module.DbHelper;
        var dt = db.ExecuteDataTable(
            $"SELECT STOCK FROM PRODUCT WHERE ITEM_ID = {MarkValue(row["ITEM_ID"])}");
        if (dt.Rows.Count == 0)
            throw new EEPException($"商品 {row["ITEM_ID"]} 不存在");

        if (Convert.ToInt32(dt.Rows[0]["STOCK"]) < (int)row["QTY"])
            throw new EEPException($"商品 {row["ITEM_ID"]} 庫存不足");
    }
}
```

### 與 `return false` 的差異

| 動作 | `return false` | `throw` |
|------|----------------|---------|
| 適用事件 | `onBefore*`（bool 回傳值的） | 任何事件 |
| 影響範圍 | **單筆跳過**（其他筆仍正常執行） | **整批 Rollback** |
| 前端提示 | 無（需自行寫 alert） | 自動顯示 EEPException 訊息 |
| `onAfterApplied` | 仍觸發（視其他筆是否成功） | 不觸發 |

## ValidateXss 允許字元完整規則

> 原始碼：`EEPServerTools.Core/Components/UpdateComponent.cs:528` 與 `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.validate.js:5`

前後端用**完全相同的 regex**：

```
^[\u4e00-\u9fa5_\uff00-\uffff_a-zA-Z0-9_`~!@#$%^&*()_\-+=?:"{}|,./;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：""【】、；''，。、\s]+$
```

### 允許

| 範圍 | 說明 |
|------|------|
| `\u4e00-\u9fa5` | 中日韓統一表意文字（基本漢字區） |
| `\uff00-\uffff` | 全形符號與字母（全形英數、全形標點、片假名半形等） |
| `a-zA-Z0-9` | 英數 |
| 半形符號 | `` ` ~ ! @ # $ % ^ & * ( ) _ - + = ? : " { } \| , . / ; ' \ [ ] `` |
| 全形標點 | `· ～ ！ @ # ￥ % …… & * （ ） —— = { } \| 《 》 ？ ： " " 【 】 、 ； ' ' ， 。` |
| `\s` | 空白字元 |

### 禁止（會被擋的常見字元）

| 字元 | 來源 | 客戶常見情境 |
|------|------|--------------|
| **越南文帶聲調**（ă â đ ê ô ơ ư 等） | U+0100-U+024F 拉丁擴充 | 越南客戶 |
| **泰文** | U+0E00-U+0E7F | 泰國客戶 |
| **韓文**（注音/半形韓） | U+AC00-U+D7AF 不完全在允許範圍 | 韓國客戶 |
| **阿拉伯文、西里爾文** 等 | U+0600+ / U+0400+ | 少見 |
| **部分拉丁變音** (ñ ü é ç 等) | U+00C0-U+017F | 西班牙/法文客戶 |
| **Emoji** | U+1F000+ | 社群類系統 |
| `< > &` | 禁止 | 任何場景 |

### 解法（前面章節已提）

- 單元件關：`validateXss: false`（DataGrid + UpdateComponent 都要）
- 全域覆寫：主頁原始碼注入 `$.validateScript = v => v`
- 客製白名單：直接改 `jquery.infolight.validate.js` 的 regex，**升級會被蓋**，記錄位置

### Server 端失敗訊息

驗證失敗會拋 `Exception`（非 `EEPException`），訊息格式為 `validateXss:{值}`。前端會攔截並轉成友善訊息（`$.fn.locale.validateXss`）。

## `UpdateIfExists` 的事件路由

啟用 `UpdateIfExists`（即「INSERT 時若主鍵已存在則改 UPDATE」）時，**事件路由會變**：

| 情境 | 觸發事件 |
|------|----------|
| INSERT + 主鍵不存在 | `onBeforeInsert` → `onAfterInsert` |
| **INSERT + 主鍵已存在（自動改 UPDATE）** | **`onBeforeUpdate` → `onAfterUpdate`**（不是 Insert 事件） |
| UPDATE | `onBeforeUpdate` → `onAfterUpdate` |
| DELETE | `onBeforeDelete` → `onAfterDelete` |

### 容易踩的坑

假設 `onBeforeInsert` 設定建立日期，`onBeforeUpdate` 設定修改日期：

```csharp
public bool uc_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    row.CREATE_DATE = DateTime.Now.ToString("yyyyMMdd");
    return true;
}

public bool uc_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    row.MODIFY_DATE = DateTime.Now.ToString("yyyyMMdd");
    return true;
}
```

若啟用 `UpdateIfExists`，**使用者以為是新增的資料**（前端按新增），但後端發現主鍵已存在→走 Update 路徑→**`CREATE_DATE` 不會被設**。

### 應對方式

在 `onBeforeUpdate` 內也補處理：

```csharp
public bool uc_onBeforeUpdate(object sender, dynamic row, List<string> sqls)
{
    // 若 CREATE_DATE 空白（表示這筆其實是 UpdateIfExists 轉來的新記錄）
    if (string.IsNullOrEmpty((string)row.CREATE_DATE))
        row.CREATE_DATE = DateTime.Now.ToString("yyyyMMdd");
    row.MODIFY_DATE = DateTime.Now.ToString("yyyyMMdd");
    return true;
}
```

或在 `onBeforeApply` 集中處理（不區分 Insert/Update）。

## 事件方法沒執行的檢查清單

客戶常卡在「我寫了事件，為什麼沒跑」。90% 是以下原因：

| 檢查項 | 說明 |
|--------|------|
| **拼字** | `_onBeforUpdate` / `_onBeforeupdate` / `_OnBeforeUpdate` 都會失敗 |
| **方法可見度** | 必須 `public`，`private` / `internal` 都跑不到（反射限制） |
| **參數簽名** | 必須是 `(object sender, dynamic row, List<string> sqls)`，少一個參數就找不到 |
| **回傳型別** | Before 事件必須 `bool`，After 事件必須 `void` |
| **所屬 class** | 必須寫在繼承 `DataModule` 的類別中（通常是 `design/server/{方案}/{模組名稱}.cs`） |
| **設計介面欄位** | UpdateComponent 的 `onBeforeInsert` 等屬性欄要**填入方法名稱** |
| **方法名稱前綴** | 通常是 `{updateComponentId}_onBefore...`，但實際上框架只比對**設計欄位中填的字串**與 C# 方法名稱 |
| **Module.InvokeMethod 反射** | 以 reflection 呼叫，找不到時**靜默失敗**（不會報錯） |

### 測試事件是否真的被呼叫

最快的方法：在事件內加一行 throw：

```csharp
public bool uc_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    throw new EEPException("測試：事件有跑到");
    return true;
}
```

若沒看到訊息 → 事件根本沒呼叫（檢查清單上的項目）。

## SQL 最終執行順序

`UpdateComponent.GetSqls()` 回傳 SQL 清單後，`DataModule.UpdateDataset()` 在 Transaction 中依下列順序執行（同一個 Transaction）：

```
[ Transaction 開始 ]
1. onBeforeApply 事件產生的 SQL
2. DELETE 明細（InfoDataSource 連動）
3. DELETE 主表各列
4. UPDATE 主表各列
5. INSERT 主表各列
6. AutoNumber.GetSqls()        ← SYSAUTONUM 回寫
7. InfoTransaction.GetSqls()    ← TRS 過帳到目標表
8. onAfterApply 事件產生的 SQL
[ 全部成功 → Commit ]
9. onAfterApplied 事件觸發（Commit 之後，不在 Transaction 內）
```

### 主明細架構時的完整順序

多個 UpdateComponent（主檔 + 各層明細）各自呼叫 `GetSqls()` 後被 `UpdateDataset` 合併，大致順序：

```
Master: onBeforeApply → Delete → Update → Insert → AutoNumber → TRS → onAfterApply
Detail: onBeforeApply → Delete → Update → Insert → AutoNumber → TRS → onAfterApply
Detail2: ...
[ Commit ]
Master onAfterApplied
```

> Detail 層級的 `onAfterApplied` **不會觸發**（[#472657](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=472657)）

## 欄位寫入權限：`ReadOnly` / `insertEnable` / `updateEnable`

三個獨立控制項共同決定欄位是否進 INSERT / UPDATE SQL。

| 控制源 | 說明 |
|--------|------|
| **Schema ReadOnly** | 資料庫層面的唯讀欄位（例如 `int IDENTITY`、計算欄位）。`DbHelper.IsReadOnly(schemaTable, colName)` 自動偵測 |
| **FieldAttr.insertEnable** | UpdateComponent 的邏輯控制，預設 `true`。設 `false` → INSERT 時跳過此欄位 |
| **FieldAttr.updateEnable** | 同上，預設 `true`。設 `false` → UPDATE 時跳過此欄位 |

### 優先順序

**AND 關係**（要「全部允許」才寫入）：

- INSERT 欄位: `!Schema.ReadOnly && FieldAttr.insertEnable`
- UPDATE 欄位: `!Schema.ReadOnly && FieldAttr.updateEnable`

### 常見用法

- **Identity 欄位** — `Schema.ReadOnly = true`，自動跳過，**不需**再設 insertEnable
- **外部表 JOIN 進來的欄位** — 設 `insertEnable = false` + `updateEnable = false`，只顯示不寫
- **建立日期** — 設 `updateEnable = false` 防止 UPDATE 時被覆蓋（但 INSERT 時填入）
- **修改日期** — 設 `insertEnable = false` + 預設 `defaultValue` 只在 Update 套用（或 DefaultMode=Update）

## `ImportMode` 值清單

`Options.ImportMode` 是字串，由前端在 Excel 匯入時設定。內建支援值：

| 值 | 用法 |
|----|------|
| `""` / null | 一般前端提交（非匯入） |
| `"replace"` | 匯入前先清空整個表（例如 `DELETE FROM USERS`）再逐筆 INSERT |
| `"insert"` | 只新增；匯入檔中若主鍵已存在，**跳過更新**（在 `onBeforeUpdate` return false） |
| `"update"`（約定俗成） | 只更新；新資料可在事件內自行判斷跳過 |
| 其他任意字串 | 純粹給事件判斷用，框架不做特殊處理 |

### 典型使用 pattern

```csharp
public void ucUser_onBeforeApply(object sender, dynamic rows, List<string> sqls)
{
    var uc = sender as UpdateComponent;
    var mode = uc.Options.ImportMode;

    if (string.IsNullOrEmpty(mode)) return;  // 非匯入情境不處理

    if (mode == "replace")
    {
        // replace 模式：先清空
        sqls.Add(uc.Module.DbHelper.GetDeleteAllSql("USERS"));
    }
    else
    {
        // insert / update / 其他：啟用 upsert 能力
        uc.UpdateIfExists = true;
    }
}
```

## 明細 SQL 失敗的 Rollback 範圍

**任一 SQL 失敗 → 整個 Transaction Rollback**。

- 主檔已成功 INSERT、明細第 5 筆失敗 → 主檔 INSERT 也退回
- AutoNumber 的 SYSAUTONUM 回寫失敗 → 業務表的資料也退回
- TRS 目標表寫入失敗 → 原表的異動也退回

**沒有**「部分成功」的模式。若需要「容錯匯入」（某筆失敗不影響其他），要在事件中用 `try-catch` 包單筆邏輯，自行決定是否 `throw`。

## `_insertedRow` 與 Identity 取值

### `_insertedRow`（內部欄位）

記錄**最新一筆 INSERT 的主表 row**（不含 Detail，排除 `Options.IsDetail = true`）。用途：

- **InfoTransaction** — TRS 取 `_insertedRow` 的 key 寫進 TargetTable
- **InfoDataSource** — 主明細關聯時取主表剛 INSERT 的 key 填入 Detail 的 `ParentValues`

### `GetInsertedValue(schemaTable, row, columnName)`

從 SQL Server 的 `SCOPE_IDENTITY()` / Oracle 的 sequence 取回剛 INSERT 產生的自動值。**只在 INSERT 之後才有意義**。

前端配合：主檔 `onApplied` reload DataGrid，系統自動把資料庫產生的 ID 帶回前端（[#470099](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=470099)）。

## `TextTransform` 執行時機

`TextTransform` 屬性（`None` / `Capitalize` / `Uppercase` / `Lowercase`）在 SQL 組裝**之前、事件之後**執行：

```
onBeforeInsert / onBeforeUpdate 事件
  ↓
TextTransform 套用（全欄位一次處理）
  ↓
FieldAttr.defaultValue 套用
  ↓
FieldAttr.trimLength 截斷
  ↓
FieldAttr.encryption 加密
  ↓
ValidateXss 驗證
  ↓
組 INSERT / UPDATE SQL
```

### 衝突情境

若 `onBeforeInsert` 寫：
```csharp
row.NAME = row.NAME.ToString().ToUpper();  // 手動轉大寫
```

同時 UpdateComponent 設 `TextTransform = Capitalize`（首字母大寫）：

- 事件先跑：`NAME = "ALICE"`
- TextTransform 再跑：`NAME = "Alice"`

**最終結果與事件寫的不同**。解法：
- 事件不要做 TextTransform 已能做的事
- 或關閉 TextTransform（設 `None`）改用事件手動控制

## 常見陷阱與限制（討論區彙整）

| 陷阱 / 限制 | 說明 | 解法 | 來源 |
|------------|------|------|------|
| **事件 SQL 非即時執行** | `sqls.Add()` 只是加入清單 | 接受此機制，不要試著在事件中直接執行 SQL | [#473877](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473877) |
| **Detail onAfterUpdate 回寫 Master 看到舊資料** | 執行順序 Master 先於 Detail 事件 | 改用 Master `onApplied` + ServerMethod | [#471374](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=471374) |
| **明細 UpdateComponent `onAfterApplied` 不觸發** | 只主檔觸發 | 需要時改在 onApplied 或 onAfterApply | [#472657](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=472657) |
| **`GetOldRow` 回傳 null** | `GetComponent` 參數寫錯（非 UpdateComponent id） | 確認參數是 UpdateComponent 的 `id` | [#482517](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482517) |
| **DataPanel 無法寫入 DB** | 與 DataForm 不同 RemoteName | 同 RemoteName + LEFT JOIN + 欄位 enable=false | [#482410](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482410) |
| **FlowFlag 欄位空白** | 流程活動 Title 屬性未填 | 填 Title | [#473801](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473801) |
| **onAfterInsert 補欄位，Flow 讀不到** | 可能不在同一 Trans | 改用 InfoCommand 虛擬欄位拼接 | [#482130](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482130) |
| **特殊字元存不進去** | DataGrid / UpdateComponent 的 validateXss 各自預設 true | 兩邊都要設 false | [#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032) |
| **Identity 自增值拿不到** | INSERT 執行中無法取得 | `onApplied` reload 後取得 | [#470099](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=470099) |
| **Exception 顯示外層訊息** | 框架優先顯示 Exception.Message | 改 `DataProvider.cs` 的 catch（非公版） | [#469106](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469106) |
| **SecColumns Encryption UI 看不到** | SP6/SP7 Collection Editor 未暴露此選項 | 直接編輯模組 JSON | [#481353](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481353) |
| **TRS TargetTable 選不到系統表** | 下拉過濾掉了 | 手動輸入表名（例如 `USERS`） | [#469257](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469257) |
| **審核中單據無法套用刪除控制** | 表單沒掛選單 / 流程參數沒設 | 檢查選單掛載 + 流程參數 + 預備模式 | [#481945](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481945) |
| **全域變數只能存字串** | `$.setVariableValue` / `ClientInfo.Variables` 是 string | 物件/陣列要先 JSON.stringify | [#474147](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474147) |

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
