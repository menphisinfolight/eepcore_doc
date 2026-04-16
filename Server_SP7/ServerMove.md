# ServerMove

> `EEPServerTools.Core/Components/ServerMove.cs` — 406 行
> 繼承：`ServerComponent` → `Component`

## 用途

**伺服器端搬移/結轉元件**（Server Move）。

ServerMove 實現 EEP Core 的「期間結轉」機制——透過 InfoCommand 查詢來源資料，再利用 InfoTransaction 產生交易 SQL，將資料搬移或累計到結算表（CloseTable）。典型應用場景包括：會計期間結帳（CarryOver）、試算結轉（CarryTrial）、期間刪除（DeletePeriod）等。設計師在模組 JSON 中以 `"type": "servermove"` 宣告。

主要提供三種操作模式：
- **Execute** — 一般搬移：依 whereStr 查詢來源資料，以 mode（1=新增 / -1=刪除）觸發 InfoTransaction 產生 SQL。
- **CarryOver** — 期間結轉：從 CloseTable 取最後一期的期末值作為新期的期初值，執行交易後計算期末值。
- **CarryTrial** — 試算結轉：逐筆依 KeyFields 計算期初/期末值，適用於更細粒度的結轉邏輯。
- **DeletePeriod** — 刪除指定期間的結算資料。

## JSON 設定範例

```json
{
  "type": "servermove",
  "id": "moveClose",
  "sourcecommand": "cmdLedger",
  "transaction": "trsLedger",
  "transscope": "All",
  "closetable": "CLOSE_BALANCE",
  "beginningfield": "BEG_AMT",
  "endingfield": "END_AMT",
  "calculation": "BEG_AMT + DR_AMT - CR_AMT",
  "keyfields": [
    { "field": "PERIOD" },
    { "field": "ACCT_NO" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **id** | string | 文字 | 元件識別碼 |
| **sourcecommand** | string | 元件選擇器 `[ControlEditor("infocommand")]` | 來源 InfoCommand（查詢資料用） |
| **transaction** | string | 元件選擇器 `[ControlEditor("infotransaction")]` | 綁定的 InfoTransaction（產生交易 SQL） |
| **transscope** | TransScope | 下拉 | 交易範圍（`All`=全部一次送出、`One`=逐筆送出） |
| **closetable** | string | 資料表選擇器 `[TableEditor]` | 結算資料表名稱 |
| **keyfields** | KeyField[] | 集合編輯器 `[CollectionEditor]` | 結算表的關鍵欄位清單 |
| **beginningfield** | string | 欄位選擇器 `[ColumnEditor("closeTable")]` | 結算表的「期初值」欄位 |
| **endingfield** | string | 欄位選擇器 `[ColumnEditor("closeTable")]` | 結算表的「期末值」欄位 |
| **calculation** | string | 文字 | 期末值計算公式（如 `BEG_AMT + DR_AMT - CR_AMT`） |

## KeyField — 關鍵欄位

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **field** | string | 欄位選擇器 `[ColumnEditor("closeTable")]` | 結算表關鍵欄位名稱 |

`KeyFields[0]` 固定作為**期間欄位**（periodField），後續欄位作為分類鍵值。

## 核心方法

### Execute(whereStr, mode)

一般搬移執行流程：

```
Execute(whereStr, mode)
  → mode 必須為 1（新增）或 -1（刪除）
  → CreateDatabaseHelper() 建立 DB 連線
  → GetSqls(whereStr, mode, "")
    → 查詢 InfoTransaction 元件
    → 查詢 InfoCommand 來源資料
    → 遍歷每筆 row → 根據 mode 組成 inserted/deleted
    → 呼叫 InfoTransaction.GetSqls() 產生交易 SQL
    → 若有明細表關聯 → 一併處理明細交易
  → TransScope == All 時直接 ExecuteSql
  → 回傳 SQL 陣列
```

### CarryOver(whereStr, thisPeriod)

期間結轉流程：

```
CarryOver(whereStr, thisPeriod)
  → 驗證 KeyFields、CloseTable 不為空
  → 查詢 CloseTable 最大期間
  → 若最大期間 == thisPeriod → 報錯（已結轉）
  → INSERT INTO CloseTable：
      以上期期末值作為新期期初值
  → GetSqls() 產生交易 SQL（mode=1）
  → 若有 Calculation → UPDATE 期初 NULL 為 0、計算期末值
  → DELETE 期末為 NULL 的無效資料
  → ExecuteSql 全部送出
```

### CarryTrial(whereStr, cField, thisPeriod)

試算結轉流程：

```
CarryTrial(whereStr, cField, thisPeriod)
  → 驗證 KeyFields >= 2、CloseTable、SourceCommand 不為空
  → 透過 InfoCommand 查詢來源資料
  → 逐筆遍歷：
    → 若分類鍵值改變 → 從 CloseTable 取上期期末作為期初
    → 否則 → 以上一筆的期末值接續計算
    → 套用 Calculation 公式計算期末值
  → ExecuteSql 全部送出
```

### DeletePeriod(thisPeriod)

刪除指定期間在 CloseTable 中的所有資料。

## DetailInfo — 明細交易資訊

GetSqls() 內部會自動偵測來源 InfoCommand 是否有明細表關聯（透過 InfoDataSource），若有，則一併觸發明細表的 InfoTransaction 交易。

| 屬性 | 類型 | 說明 |
|------|------|------|
| **Relation** | InfoDataSource | 主明細關聯 |
| **Transation** | InfoTransaction | 明細表的交易元件 |

## TransScope — 交易範圍

| 模式 | 說明 |
|------|------|
| **All** | 所有 SQL 一次送出（預設） |
| **One** | 逐筆資料各自送出（有明細表時自動切換為此模式） |

## 備註

- ServerMove 本身不直接產生交易 SQL，而是透過綁定的 InfoTransaction 元件來產生。
- `mode` 參數只接受 `1`（視為新增）或 `-1`（視為刪除），用於控制交易方向。
- 程式碼中 GetSqls() 的 `mode == -1` 分支存在 bug：第 350 行和第 370 行的條件判斷都寫成了 `mode == 1`，導致 `mode == -1` 時不會將資料放入 `deleted`，而是無操作。
- CarryOver 會先檢查是否重複結轉（同一期間不可結轉兩次）。
- CarryTrial 適用於需要逐筆累計的場景（如會計科目試算表），KeyFields 至少需要兩個欄位（期間 + 分類）。
- CreateDatabaseHelper() 會使用 InfoCommand 指定的 Database（若有設定），否則使用預設資料庫。
- 執行完畢後透過 ResetDatabaseHelper() 還原 Module.DbHelper，避免影響其他元件。
