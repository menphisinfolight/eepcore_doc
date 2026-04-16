# InfoTransaction

> `EEPServerTools.Core/Components/InfoTransaction.cs` — 679 行
> 繼承：`UpdateBaseComp` → `ServerComponent` → `Component`

## 用途

**交易回寫元件**（Info Transaction）。

InfoTransaction 實現 EEP Core 的「資料表交易」機制——當原始資料表發生 INSERT/UPDATE/DELETE 時，自動觸發對目標資料表的欄位回寫（累加、累減、替換、回寫等）。設計師在模組 JSON 中以 `"type": "infotransaction"` 宣告，無需寫程式碼即可實現跨表的資料同步。

搭配 [SYS_TRSFILES](../EEP%20Core系統資料表/SYS_TRSFILES.md) 儲存交易定義。

## JSON 設定範例

```json
{
  "type": "infotransaction",
  "id": "trsOrderToCustomer",
  "updatecomponent": "ucOrder",
  "transactions": [{
    "targetTable": "CUSTOMER",
    "transMode": "AutoAppend",
    "whenInsert": true,
    "whenUpdate": true,
    "whenDelete": true,
    "keyFields": [
      { "sourceField": "CUST_NO", "targetField": "CUST_NO" }
    ],
    "fields": [
      { "sourceField": "AMOUNT", "targetField": "TOTAL_AMOUNT", "updateMode": "Increase" },
      { "sourceField": "ORDER_DATE", "targetField": "LAST_ORDER_DATE", "updateMode": "Replace" }
    ]
  }]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **id** | string | 文字 | 元件識別碼 |
| **updatecomponent** | string | 元件選擇器 `[ControlEditor]` | 綁定的 UpdateComponent ID（來源資料表） |
| **transactions** | Transaction[] | 集合編輯器 `[CollectionEditor]` | 交易規則清單（可多個目標表） |

## Transaction — 交易規則

每個 Transaction 定義一條交易規則（原始表 → 一個目標表）：

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **targetTable** | string | 資料表選擇器 `[TableEditor]` | — | 目標資料表名稱 |
| **transMode** | TransMode | 下拉 | — | 交易模式（見下方列舉） |
| **whenInsert** | bool | 勾選框 | true | INSERT 時觸發 |
| **whenUpdate** | bool | 勾選框 | true | UPDATE 時觸發 |
| **whenDelete** | bool | 勾選框 | true | DELETE 時觸發 |
| **keyFields** | KeyField[] | 集合編輯器 | — | 關聯鍵值（原始表↔目標表的欄位對應） |
| **fields** | Field[] | 集合編輯器 | — | 回寫欄位（見下方） |
| **exceptionMessage** | string | 文字 | — | 失敗時的錯誤訊息 |
| **batchProcess** | bool | 勾選 | false | 是否為批次處理模式 |
| **autoNumber** | string | 元件選擇器 | — | 搭配的 AutoNumber 元件（目標表自動編號） |
| **onBeforeTrans** | string | 事件編輯器 | — | 交易前事件（return false 可跳過） |

## TransMode — 交易模式

| 模式 | 說明 | 目標表已存在時 | 目標表不存在時 |
|------|------|--------------|--------------|
| **AutoAppend** | 自動新增 | UPDATE | INSERT |
| **AlwaysAppend** | 永遠新增 | INSERT（新增一筆） | INSERT |
| **SyncAppend** | 同步新增 | UPDATE（有舊值時）/ INSERT（無舊值時） | INSERT；DELETE 時刪除目標 |
| **Ignore** | 忽略不存在 | UPDATE | 不處理 |
| **Exception** | 必須存在 | UPDATE | 報錯（RowsAffectCheck） |

## UpdateMode — 回寫方式

| 模式 | 符號 | SQL 產生結果 | 說明 |
|------|------|-------------|------|
| **Increase** | + | `target = ISNULL(target,0) + (newValue - oldValue)` | 累加 |
| **Decrease** | - | `target = ISNULL(target,0) - (newValue - oldValue)` | 累減 |
| **DecreaseNotZero** | -x | 同 Decrease，額外加 `WHERE target >= value` | 累減不可為負 |
| **Replace** | = | `target = newValue` | 替換 |
| **ReplaceNegative** | =- | `target = 0 - value` | 替換為負值 |
| **WriteBack** | <= | `source.field = target.field` | 回寫（目標→原始） |
| **WriteBackInc** | <+ | `source.field = source.field + target.field` | 回寫累加 |
| **WriteBackDec** | <- | `source.field = source.field - target.field` | 回寫累減 |

### Increase/Decrease 的新舊值差異計算

UPDATE 時，交易不是直接加上新值，而是計算**新舊值差異**：

```
累加：target = target + (newValue - oldValue)
累減：target = target - (newValue - oldValue)
```

這確保了 UPDATE 時只加/減變動量，而非重複累計。

### 鍵值變更處理

如果 UPDATE 時關聯鍵值也被改了（如客戶編號從 A 改成 B），系統會：
1. 在新鍵對應的目標行加上新值
2. 在舊鍵對應的目標行減去舊值

## Field — 回寫欄位

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **targetField** | string | 欄位選擇器 `[ColumnEditor("target")]` | 目標表欄位 |
| **sourceField** | string | 欄位選擇�� `[ColumnEditor]` | 來源表欄位 |
| **sourceValue** | string | 預設值編輯器 `[ValueOptionEditor]` | 來源值（替代 sourceField） |
| **updateMode** | UpdateMode | 下拉 | 回寫方式（見上方列舉） |

### SourceValue 類型

sourceValue 支援比 UpdateComponent 更多的值類型：

| 類型 | 格式 | 說明 |
|------|------|------|
| `constant` | `constant["值"]` | 固定值 |
| `varaible` | `varaible["today"]` | 變數（user/today/now/lastkey/period 等） |
| `function` | `function["方法名"]` | 呼叫自訂方法 |
| `parent` | `parent["欄位名"]` | 從主表查詢欄位值（透過 InfoDataSource） |
| `sql` | `sql["表達式"]` | SQL 表達式，`[欄位名]` 會被替換為行值 |

## KeyField — 關聯鍵值

| 屬性 | 類型 | 說明 |
|------|------|------|
| **targetField** | string | 目標表關聯欄位 |
| **sourceField** | string | 來源表關聯欄位（特殊值：`$LASTKEY` = AutoNumber 最新值、`$PERIOD` = 期間值） |

## GetSqls() 流程

```
GetSqls(rows)
  → 遍歷每個 Transaction
    │
    ├─ deleted（whenDelete = true）
    │   → 每筆 row → GetSqls(tran, schema, null, row)
    │     → OnBeforeTrans(null, row) → false 則跳過
    │     → 根據 TransMode 產生 SQL
    │
    ├─ updated（whenUpdate = true）
    │   → 每筆 row → 先查 oldRow（command.GetOldRow）
    │   → GetSqls(tran, schema, row, oldRow)
    │     → 計算新舊值差異
    │
    ├─ inserted（whenInsert = true）
    │   → 每筆 row → GetSqls(tran, schema, row, null)
    │
    └─ AutoNumber.GetSqls()（回寫流水號）

每筆 row 的 SQL 產生：
  1. 根據 TransMode 產生主要 SQL（UPDATE/INSERT/IF-ELSE）
  2. 若有鍵值變更 → 額外產生舊鍵的反向 SQL
  3. 若有 WriteBack 欄位 → 產生回寫 SQL（UPDATE 原始表）
```

## 實際範例

### 出貨單 → 客戶累計營收

```
原始表：ORDER（出貨單）
目標表：CUSTOMER（客戶）
關聯：ORDER.CUST_NO → CUSTOMER.CUST_NO
回寫：ORDER.AMOUNT → CUSTOMER.TOTAL_AMOUNT（Increase）

新增出貨單（AMOUNT=1000）：
  → UPDATE CUSTOMER SET TOTAL_AMOUNT = ISNULL(TOTAL_AMOUNT,0) + (1000 - 0)
    WHERE CUST_NO = 'C001'

修改出貨單（AMOUNT 從 1000 改為 1500）：
  → UPDATE CUSTOMER SET TOTAL_AMOUNT = ISNULL(TOTAL_AMOUNT,0) + (1500 - 1000)
    WHERE CUST_NO = 'C001'

刪除出貨單（AMOUNT=1500）：
  → UPDATE CUSTOMER SET TOTAL_AMOUNT = ISNULL(TOTAL_AMOUNT,0) + (0 - 1500)
    WHERE CUST_NO = 'C001'
```

## 備註

- InfoTransaction 由 UpdateComponent.GetSqls() 自動觸發（找到所有 `Updatecomponent == Id` 的 InfoTransaction）。
- 刪除主表時，InfoDataSource.GetDeleteDetailSqls() 也會觸發明細表的交易回寫。
- DecreaseNotZero 模式會在 WHERE 中加入 `target >= value` 條件，若不符合則 RowsAffectCheck 報錯（防止庫存為負）。
- WriteBack 系列（<=、<+、<-）方向相反：從**目標表**寫回**原始表**，使用 `UPDATE source FROM target WHERE ...` 語法。
- `$LASTKEY` 特殊值取自 AutoNumber 最新產生的值，用於 AlwaysAppend 模式下目標表的自動編號。
