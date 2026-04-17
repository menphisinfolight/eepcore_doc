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
