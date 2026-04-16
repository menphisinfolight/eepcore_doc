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

```csharp
// 簽名：string fn(sender, fixedString, rows)
// 可程式化修改前綴
public string autoOrderNo_onGetFixed(object sender, string fixedString, dynamic rows)
{
    return fixedString + "A"; // 加上自訂後綴
}
```

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
