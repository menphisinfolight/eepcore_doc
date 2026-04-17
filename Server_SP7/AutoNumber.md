# AutoNumber

> `EEPServerTools.Core/Components/AutoNumber.cs` — 196 行
> 繼承：`ServerComponent` → `Component`

## 用途

**自動編號元件**（Auto Number）。

AutoNumber 在資料新增時自動產生不重複的流水號編號。搭配 [SYSAUTONUM](../EEP%20Core系統資料表/SYSAUTONUM.md) 系統資料表管理流水號狀態。設計師在模組 JSON 中以 `"type": "autonumber"` 宣告，綁定到 UpdateComponent，指定目標欄位和編號格式。

## JSON 設定範例

```json
{
  "type": "autonumber",
  "id": "autoOrderNo",
  "autoNoID": "OrderNo",
  "updatecomponent": "ucOrder",
  "field": "ORDER_NO",
  "getFixed": "_yyyyMM",
  "startValue": 1,
  "step": 1,
  "numDig": 4
}
```

產生結果：`202604 0001`、`202604 0002`...（202604 + 4 位流水號）

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **autoNoID** | string | 文字 | — | 編號規則 ID（對應 SYSAUTONUM.AUTOID） |
| **updatecomponent** | string | 元件選擇器 `[ControlEditor]` | — | 綁定的 UpdateComponent ID |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 目標欄位名稱（必填） |
| **description** | string | 文字 | — | 描述 |
| **getFixed** | string | 文字 | "" | 前綴格式（見下方說明） |
| **startValue** | long | 數字框 `[NumberboxEditor]` | 1 | 流水號起始值 |
| **step** | int | 數字框 `[NumberboxEditor]` | 1 | 遞增步進 |
| **numDig** | int | 數字框 `[NumberboxEditor]` | 3 | 流水號位數（不足補零） |
| **readOnly** | bool | 勾選 | false | 防止使用者手動編輯自動編號欄位 |
| **onGetFixed** | string | 事件編輯器 `[EventEditor]` | — | 自訂前綴事件 |

## GetFixed 前綴格式

| 格式 | 範例 | 說明 |
|------|------|------|
| 空字串 | `001`, `002` | 無前綴，純流水號 |
| `ORD` | `ORD001`, `ORD002` | 固定字串前綴 |
| `_yyyyMM` | `202604001` | 日期格式（年月） |
| `_yyyyMMdd` | `20260416001` | 日期格式（年月日） |
| `_'INV'yyyyMM` | `INV202604001` | 固定字串 + 日期格式 |

### 格式解析規則

`_` 開頭表示使用日期格式：
- `_` 後面用單引號包住的是固定字串部分（如 `'INV'`）
- 剩餘部分是 `DateTime.Now.ToString()` 的格式字串（如 `yyyyMM`）

### OnGetFixed 事件

當 `getFixed` 屬性不夠彈性時，用這個事件動態計算前綴（例如要依客戶、部門、單據欄位組合）。

**簽名**：

```csharp
string fn(object sender, string fixedString, dynamic rows)
```

| 參數 | 型別 | 說明 |
|------|------|------|
| `sender` | `object` | AutoNumber 實例，需 cast：`(sender as AutoNumber)` |
| `fixedString` | `string` | `getFixed` 屬性解析後的預設前綴（例如 `"202604"`），若 `getFixed` 為空則為 `""` |
| `rows` | `dynamic` | 要 INSERT 的 JArray（可存取第一筆的欄位：`rows[0].欄位名` 或 `rows[0]["欄位名"]`） |
| 回傳 | `string` | 最終要用的前綴字串（會與流水號串接） |

**觸發時機**：`AutoNumber.Init()` 呼叫一次，在 SELECT SYSAUTONUM 查詢當前流水號**之前**。前綴不同會對應到 SYSAUTONUM 中不同的 `FIXED` 列，各自獨立計數。

#### 範例 1：客戶編號 + 年月（每客戶每月獨立流水）

```csharp
public string an出貨單_onGetFixed(object sender, string fixedString, dynamic rows)
{
    // 取得目前的年月，格式為 yyyyMM (例如：202603)
    string datePart = DateTime.Now.ToString("yyyyMM");

    // 組合：客戶編號 + "_" + 年月
    fixedString = rows[0].客戶編號 + "_" + datePart;

    return fixedString;
}
```

產生結果：`C001_202604-0001`、`C001_202604-0002`、`C002_202604-0001`（C001 和 C002 各自獨立）

#### 範例 2：依部門代碼動態前綴

```csharp
public string an採購單_onGetFixed(object sender, string fixedString, dynamic rows)
{
    var clientInfo = (sender as AutoNumber).Module.ClientInfo;
    string deptCode = (string)rows[0].DEPT_CODE;
    string yearMonth = DateTime.Now.ToString("yyyyMM");

    // PO-部門-年月：PO-HR-202604、PO-IT-202604
    return $"PO-{deptCode}-{yearMonth}";
}
```

#### 範例 3：依單據類型前綴（條件式）

```csharp
public string an訂單_onGetFixed(object sender, string fixedString, dynamic rows)
{
    string orderType = (string)rows[0].ORDER_TYPE;
    string yearMonth = DateTime.Now.ToString("yyyyMM");

    // 內銷用 D、外銷用 E 開頭
    string prefix = orderType == "EXPORT" ? "E" : "D";
    return $"{prefix}{yearMonth}";
}
```

產生結果：`D202604-001`（內銷）、`E202604-001`（外銷）

#### 範例 4：民國年 + 週次

```csharp
public string an週報_onGetFixed(object sender, string fixedString, dynamic rows)
{
    var now = DateTime.Now;
    int rocYear = now.Year - 1911;  // 民國年
    int weekOfYear = System.Globalization.ISOWeek.GetWeekOfYear(now);

    // 格式：113W16（民國113年第16週）
    return $"{rocYear:D3}W{weekOfYear:D2}";
}
```

#### 範例 5：查資料庫動態取前綴

```csharp
public string an發票_onGetFixed(object sender, string fixedString, dynamic rows)
{
    var db = (sender as AutoNumber).Module.DbHelper;
    var clientInfo = (sender as AutoNumber).Module.ClientInfo;

    // 查使用者所屬公司的發票字軌
    var dt = db.ExecuteDataTable(
        $"SELECT TAX_PREFIX FROM COMPANY WHERE COMPANY_ID = {db.MarkValue((object)clientInfo.Solution)}"
    );
    string taxPrefix = dt.Rows.Count > 0 ? dt.Rows[0]["TAX_PREFIX"].ToString() : "AA";

    // 每兩個月一期（發票字軌 + 期別）
    int period = ((DateTime.Now.Month - 1) / 2) + 1;
    return $"{taxPrefix}{DateTime.Now:yy}{period:D2}";
}
```

產生結果：`AA2602-001`（AA 字軌、2026 年第 2 期）

#### 注意事項

- 回傳的 `fixedString` 不同 → SYSAUTONUM 中會是不同列，各自獨立計數
- 若要**每天**重置編號，回傳含 `yyyyMMdd`；若要**每月**重置，含 `yyyyMM`
- 若 `rows[0].欄位` 可能為 null，記得判空處理
- `rows` 是 `JArray`，批次 INSERT 多筆時通常只用 `rows[0]` 當作群組依據（同一批假設同前綴）

## 運作流程

```
UpdateComponent.GetSqls() 觸發
│
├─ 1. Init(insertedRows)
│     → GetFixedString() 解析 GetFixed 產生前綴
│     → 觸發 OnGetFixed 事件
│     → SELECT * FROM SYSAUTONUM WHERE AUTOID=@id AND FIXED=@fixed
│     → 有記錄 → _currentValue = CURRNUM
│     → 無記錄 → _currentValue = StartValue
│
├─ 2. GetValue()（每筆 INSERT 行呼叫一次）
│     → value = _currentValue
│     → _currentValue += Step
│     → 組合：前綴 + 補零 + 流水號
│     → 回傳編號字串
│     → 寫入 row[Field]
│
└─ 3. GetSqls()（全部 INSERT 處理完後）
      → 有記錄（_initValue 有值）：
      │   UPDATE SYSAUTONUM SET CURRNUM=@new
      │   WHERE AUTOID=@id AND FIXED=@fixed AND CURRNUM=@old
      │   （樂觀鎖：CURRNUM=@old 防止並發衝突）
      │
      → 無記錄：
          INSERT INTO SYSAUTONUM (AUTOID, FIXED, CURRNUM) VALUES (...)
```

## 樂觀鎖機制

UPDATE SYSAUTONUM 時的 WHERE 條件包含 `CURRNUM = @initValue`：

```sql
UPDATE SYSAUTONUM SET CURRNUM = 5
WHERE AUTOID = 'OrderNo' AND FIXED = '202604' AND CURRNUM = 1
```

如果另一個交易已經改了 CURRNUM（如從 1 改成 3），這條 UPDATE 會影響 0 筆，配合 `RowsAffectCheck` 就會報錯，避免編號重複。

## 編號格式範例

| autoNoID | getFixed | numDig | startValue | 產生結果 |
|----------|----------|--------|-----------|---------|
| OrderNo | 空 | 3 | 1 | `001`, `002`, `003` |
| OrderNo | `_yyyyMM` | 4 | 1 | `2026040001`, `2026040002` |
| InvNo | `_'INV'yyyyMM` | 3 | 1 | `INV202604001`, `INV202604002` |
| SeqNo | `_'A'` | 5 | 100 | `A00100`, `A00101` |

## 在 C# 中手動取號（不使用 UpdateComponent）

當你用 C# 自己寫 INSERT（例如 API、`onBeforeApply` 事件中、客製 Action），又要維持 SYSAUTONUM 的一致性時，**不要自己查 SYSAUTONUM + 自己 UPDATE** — 應該透過 AutoNumber 元件實例來取號。

### 核心 API 三步驟

```csharp
var autoNumber = Container.GetComponent<AutoNumber>("your_autoNumber_id");

autoNumber.Init(rows);              // 1. 查 SYSAUTONUM 讀取目前 CURRNUM（必須先呼叫）
var no = autoNumber.GetValue();     // 2. 取下一個編號（例如 "202604001"），內部 _currentValue += Step
var sqls = autoNumber.GetSqls();    // 3. 產生 UPDATE/INSERT SYSAUTONUM 的 SQL（樂觀鎖）
```

| 步驟 | 必要性 | 說明 |
|------|--------|------|
| `Init(rows)` | **必要，且只能一次** | 觸發 `OnGetFixed` 事件、計算 `_fixedString`、SELECT SYSAUTONUM 取得當前 CURRNUM |
| `GetValue()` | 每筆一次 | 回傳格式化後的編號字串，內部流水號累加（`_currentValue += Step`） |
| `GetSqls()` | **必要（否則 SYSAUTONUM 不會更新）** | 產出 1 條 UPDATE 或 INSERT SQL，**必須在同一 Transaction 中執行** |

### 情境 1：在 `onBeforeApply` 事件中手動取號

```csharp
public void uc訂單_onBeforeApply(object sender, dynamic rows, List<string> sqls)
{
    var module = (sender as UpdateComponent).Module;

    // 取得 AutoNumber 元件實例
    var autoNumber = module.GetComponent<AutoNumber>("an訂單");

    // 初始化（傳入 inserted rows 讓 OnGetFixed 能存取欄位）
    var inserted = (JArray)rows["inserted"];
    autoNumber.Init(inserted);

    // 對每筆 INSERT 取號並寫回欄位
    foreach (JObject row in inserted)
    {
        row["ORDER_NO"] = autoNumber.GetValue();
    }

    // 將 SYSAUTONUM 更新 SQL 加入同一 Transaction
    sqls.AddRange(autoNumber.GetSqls());
}
```

### 情境 2：請購單轉採購單（客製 API / Action）

實務場景：使用者在請購單（REQUISITION）按下「轉採購」按鈕，後端需要：

1. 撈取指定請購單的抬頭與明細
2. 用 AutoNumber 取採購單號
3. 寫入採購單（PURCHASE_ORDER）抬頭與明細
4. 更新請購單狀態為「已轉採購」
5. 全部在同一 Transaction，任何一步失敗全部 Rollback

```csharp
public object 轉採購(string reqNo)
{
    var dataModule = new DataModule(ClientInfo, "採購模組");
    var autoNumber = dataModule.GetComponent<AutoNumber>("an採購單");

    using (var db = dataModule.CreateDatabaseHelper(ClientInfo.Database, DatabaseType.Normal, transaction: true))
    {
        try
        {
            // ===== 1. 撈取請購單抬頭 =====
            var reqHead = db.ExecuteDataTable(
                $"SELECT * FROM REQUISITION WHERE REQ_NO = {db.MarkValue((object)reqNo)}");
            if (reqHead.Rows.Count == 0)
                throw new EEPException($"請購單 {reqNo} 不存在");

            var status = reqHead.Rows[0]["STATUS"]?.ToString();
            if (status == "TRANSFERRED")
                throw new EEPException($"請購單 {reqNo} 已轉採購過，不可重複");

            // ===== 2. 撈取請購單明細 =====
            var reqDetails = db.ExecuteDataTable(
                $"SELECT * FROM REQUISITION_DETAIL WHERE REQ_NO = {db.MarkValue((object)reqNo)}");
            if (reqDetails.Rows.Count == 0)
                throw new EEPException($"請購單 {reqNo} 無明細資料");

            // ===== 3. AutoNumber 取採購單號 =====
            // 包成 JArray 傳給 Init，供 OnGetFixed 事件存取抬頭欄位
            var headRows = new JArray { JObject.FromObject(reqHead.Rows[0].ItemArray
                .Select((v, i) => new { k = reqHead.Columns[i].ColumnName, v })
                .ToDictionary(x => x.k, x => x.v)) };

            autoNumber.Init(headRows);
            var poNo = autoNumber.GetValue();  // 例如 "PO202604-001"

            // ===== 4. 組採購單抬頭 INSERT =====
            var sqls = new List<string>();
            sqls.Add($@"
                INSERT INTO PURCHASE_ORDER 
                    (PO_NO, SUPPLIER_ID, REQ_NO, CREATE_USER, CREATE_DATE, STATUS)
                VALUES (
                    {db.MarkValue((object)poNo)},
                    {db.MarkValue(reqHead.Rows[0]["SUPPLIER_ID"])},
                    {db.MarkValue((object)reqNo)},
                    {db.MarkValue((object)ClientInfo.User)},
                    {db.MarkValue((object)DateTime.Now.ToString("yyyyMMdd"))},
                    'DRAFT'
                )");

            // ===== 5. 組採購單明細 INSERT（逐筆） =====
            int seq = 1;
            foreach (DataRow d in reqDetails.Rows)
            {
                sqls.Add($@"
                    INSERT INTO PURCHASE_ORDER_DETAIL
                        (PO_NO, SEQ_NO, ITEM_CODE, QTY, UNIT_PRICE, AMOUNT, REQ_NO, REQ_SEQ)
                    VALUES (
                        {db.MarkValue((object)poNo)},
                        {db.MarkValue((object)(seq++).ToString())},
                        {db.MarkValue(d["ITEM_CODE"])},
                        {db.MarkValue(d["QTY"])},
                        {db.MarkValue(d["UNIT_PRICE"])},
                        {db.MarkValue(d["AMOUNT"])},
                        {db.MarkValue((object)reqNo)},
                        {db.MarkValue(d["SEQ_NO"])}
                    )");
            }

            // ===== 6. 更新請購單狀態（樂觀鎖：WHERE STATUS=...）=====
            sqls.Add($@"
                UPDATE REQUISITION 
                SET STATUS = 'TRANSFERRED',
                    PO_NO = {db.MarkValue((object)poNo)},
                    TRANSFER_DATE = {db.MarkValue((object)DateTime.Now.ToString("yyyyMMdd"))}
                WHERE REQ_NO = {db.MarkValue((object)reqNo)}
                  AND STATUS <> 'TRANSFERRED'");

            // ===== 7. AutoNumber 回寫 SYSAUTONUM =====
            sqls.AddRange(autoNumber.GetSqls());

            // ===== 8. 執行（RowsAffectCheck 讓 UPDATE 0 筆時拋錯，保障樂觀鎖）=====
            db.ExecuteNonQuery(sqls, new UpdateOptions { RowsAffectCheck = true });

            db.Transaction.Commit();
            return new { poNo, reqNo, detailCount = reqDetails.Rows.Count };
        }
        catch
        {
            db.Transaction.Rollback();
            throw;
        }
    }
}
```

**這個範例示範的重點**：

- AutoNumber 取號、新表 INSERT、舊表 UPDATE、SYSAUTONUM 回寫 **全部在同一 Transaction**
- `RowsAffectCheck = true` 同時保護兩件事：
  - SYSAUTONUM 樂觀鎖（防號碼重複）
  - 請購單狀態樂觀鎖（`STATUS <> 'TRANSFERRED'` 防重複轉單）
- 業務檢核（已轉採購、無明細）在 Transaction 開始後、SQL 執行前先 throw，避免做無用功
- 若 AutoNumber 的 `OnGetFixed` 事件需要用請購單的資料（如客戶編號），透過 `Init(headRows)` 傳入即可在事件中用 `rows[0].CUSTOMER_ID` 存取

### 情境 3：批次取多個號

一次 Init，多次 GetValue（每次累加 Step），最後一次 GetSqls：

```csharp
public object 批次建立訂單(JArray orders)
{
    var dataModule = new DataModule(ClientInfo, "訂單模組");
    var autoNumber = dataModule.GetComponent<AutoNumber>("an訂單");

    using (var db = dataModule.CreateDatabaseHelper(ClientInfo.Database, DatabaseType.Normal, true))
    {
        try
        {
            autoNumber.Init(orders);  // 只呼叫一次

            var sqls = new List<string>();
            foreach (JObject order in orders)
            {
                order["ORDER_NO"] = autoNumber.GetValue();  // 每筆各自取號
                sqls.Add($"INSERT INTO ORDERS ... VALUES ({db.MarkValue(order["ORDER_NO"])}, ...)");
            }
            sqls.AddRange(autoNumber.GetSqls());  // 最後一次，一筆 UPDATE 就把 CURRNUM 推到最終值

            db.ExecuteNonQuery(sqls, new UpdateOptions { RowsAffectCheck = true });
            db.Transaction.Commit();
            return new { count = orders.Count };
        }
        catch
        {
            db.Transaction.Rollback();
            throw;
        }
    }
}
```

### 為什麼不要自己查 SYSAUTONUM？

**錯誤作法**：

```csharp
// ❌ 這樣做有並發衝突風險
var dt = db.ExecuteDataTable("SELECT CURRNUM FROM SYSAUTONUM WHERE AUTOID='OrderNo'");
var nextNum = (int)dt.Rows[0]["CURRNUM"] + 1;
db.ExecuteNonQuery($"UPDATE SYSAUTONUM SET CURRNUM = {nextNum} WHERE AUTOID='OrderNo'");
```

問題：
1. **並發衝突** — 兩個請求同時 SELECT 得到同值，UPDATE 也都成功 → 兩邊各拿到相同號碼
2. **沒處理 FIXED 欄位** — 年月前綴、客戶前綴都要額外處理
3. **沒處理首次建立** — SYSAUTONUM 無記錄時要 INSERT，不是 UPDATE
4. **跳過 OnGetFixed 事件** — 失去自訂前綴邏輯

**AutoNumber 的做法**：

```sql
-- 有記錄時：樂觀鎖 UPDATE（WHERE 條件含 CURRNUM = @old）
UPDATE SYSAUTONUM SET CURRNUM = 6
WHERE AUTOID = 'OrderNo' AND FIXED = '202604' AND CURRNUM = 5
-- 若另一交易已改 CURRNUM，此 UPDATE 影響 0 筆 → RowsAffectCheck 拋錯 → Rollback

-- 無記錄時：INSERT 新流水
INSERT INTO SYSAUTONUM (AUTOID, FIXED, CURRNUM) VALUES ('OrderNo', '202604', 2)
```

### 關鍵注意事項

1. **`RowsAffectCheck = true` 很重要** — 否則樂觀鎖失效，會產生重複號碼
2. **`GetSqls()` 一定要在同一 Transaction** — 若 INSERT 主資料成功但 SYSAUTONUM 沒更新，下次取號會拿到相同值
3. **`Init()` 只能呼叫一次** — 重複呼叫會重置 `_currentValue`，之前 `GetValue()` 的累加會遺失
4. **`GetValue()` 若超出 numDig 位數** — 不會截斷也不會報錯，只是前綴補零失效（例如 numDig=3 但值到 1500 → 產生 `1500` 四位數）
5. **`Field` 屬性若未設定** — `GetValue()` 會拋 `EEPException`
6. **Informix 的處理** — AutoNumber 內部已判斷 Connection 字串自動切換 SQL，手動呼叫時不用額外處理

### 錯誤範例

```csharp
// ❌ 沒呼叫 Init → _currentValue 是 0，結果從 StartValue 開始算但不會查 SYSAUTONUM
autoNumber.GetValue();

// ❌ 漏呼叫 GetSqls → SYSAUTONUM 永遠停在初始值，下次取號重複
autoNumber.Init(rows);
row["NO"] = autoNumber.GetValue();
// forgot autoNumber.GetSqls()!

// ❌ 沒有 RowsAffectCheck → 樂觀鎖失效
db.ExecuteNonQuery(sqls);  // 缺 UpdateOptions { RowsAffectCheck = true }

// ❌ GetSqls 不在 Transaction 中 → 失敗時主表已 INSERT 但 SYSAUTONUM 沒更新
```

## 與其他元件的關係

```
AutoNumber
  ├── 綁定 → UpdateComponent（透過 updatecomponent 屬性）
  │   └── INSERT 時自動觸發 Init() + GetValue()
  ├── 搭配 → InfoTransaction（透過 transaction.autoNumber 屬性）
  │   └── 交易回寫時為目標表產生編號
  └── 儲存 → SYSAUTONUM（系統資料表）
      └── 以 AUTOID + FIXED 為鍵，管理流水號狀態
```

## 備註

- AutoNumber 的 SQL 在 UpdateComponent.GetSqls() 的最後才執行（先 INSERT 資料行，再回寫 SYSAUTONUM），確保在同一 Transaction 中。
- 同一 AUTOID 可依不同 FIXED（如月份）維護獨立流水號，每月自動重置。
- 批次 INSERT 多筆時，Init() 只呼叫一次，GetValue() 每筆呼叫一次，CURRNUM 一次回寫最終值。
- Informix 有特殊的表名/欄位名引號處理（`DbHelper.MarkColumn("SYSAUTONUM")`）。
