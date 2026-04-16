# InfoDataSource

> `EEPServerTools.Core/Components/InfoDataSource.cs` — 228 行
> 繼承：`SelectComp` → `ServerComponent` → `Component`

## 用途

**主從資料來源元件**（Info Data Source / Master-Detail Relation）。

InfoDataSource 定義兩個 InfoCommand 之間的主從關聯。設計師在模組 JSON 中以 `"type": "infodatasource"` 宣告，指定主表命令、明細命令、以及關聯欄位。Runtime 時負責：

1. **查詢明細** — 根據主表選取的行，產生明細表的 WHERE 條件
2. **刪除連動** — 刪除主表時自動刪除所有關聯明細（遞迴）
3. **父值注入** — INSERT 明細時，自動從主表帶入關聯欄位值（含 Identity/AutoNumber）

## JSON 設定範例

```json
{
  "type": "infodatasource",
  "id": "idsMenuUser",
  "masterCommand": "menu",
  "detailCommand": "menuUser",
  "masterColumns": "MENUID",
  "detailColumns": "MENUID"
}
```

多欄位關聯：

```json
{
  "type": "infodatasource",
  "id": "idsOrderDetail",
  "masterCommand": "order",
  "detailCommand": "orderDetail",
  "masterColumns": "ORDER_NO,ORDER_DATE",
  "detailColumns": "ORDER_NO,ORDER_DATE"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **id** | string | 文字 | 元件識別碼（唯一） |
| **masterCommand** | string | 元件選擇器 `[ControlEditor]` | 主表的 InfoCommand ID |
| **masterColumns** | string | 欄位選擇器 `[ColumnEditor("master")]` | 主表關聯欄位（逗號分隔） |
| **detailCommand** | string | 元件選擇器 `[ControlEditor]` | 明細表的 InfoCommand ID |
| **detailColumns** | string | 欄位選擇器 `[ColumnEditor("detail")]` | 明細表關聯欄位（逗號分隔） |

## 核心方法

### GetDetailSql() — 查詢明細

由 `DataModule.GetDataset()` 呼叫，產生明細表的查詢 SQL。

```
GetDetailSql()
  → 取得 detailCommand（InfoCommand）
  → 設定 SecColumns（欄位安全）
  → 設定 SecStrs（列安全）
  → 加入 WHERE：detailColumn = parentRow[masterColumn]
  → 觸發 detailCommand.OnBeforeExecuteSQL
  → 回傳 SQL
```

**無主表選取時**的行為：
- `Options.AllData = true` → 回傳全部明細（不加 WHERE）
- 否則 → 加入 `WHERE 1=0`（不回傳任何資料）

### GetDeleteDetailSqls() — 刪除連動

由 `UpdateComponent.GetDeleteSqls()` 呼叫，刪除主表時自動刪除關聯明細。

```
GetDeleteDetailSqls(parentRow)
  → 遞迴：若明細表也有下層明細 → 先刪下層
  → DELETE FROM {detailTable} WHERE {detailColumn} = {parentRow[masterColumn]}
  → 若明細表有 InfoTransaction → 觸發交易回寫（反向沖銷）
```

**遞迴刪除**：支援多層主從（主→明細→子明細），自動向下遞迴刪除。

**交易回寫整合**：刪除明細前會查出所有明細資料，傳給 InfoTransaction 產生反向沖銷 SQL。

### GetParentValues() — 父值注入

由 `DataModule.UpdateDataset()` 呼叫，INSERT 明細時從主表帶入關聯欄位值。

```
GetParentValues()
  → 找到主表的 UpdateComponent
  → 遍歷 masterColumns / detailColumns
  → 從 updateComponent.GetInsertedValue(masterColumn) 取值
    （含 Identity 欄位的自動產生值）
  → 若主表也是別人的明細 → 遞迴向上取值
  → 回傳 { detailColumn: value }
```

**解決的問題**：主表是新增（INSERT）且主鍵為 Identity 自動遞增時，明細表需要等主表 INSERT 完成後才能取得主鍵值。`GetParentValues()` 透過 `GetInsertedValue()` 取得 Identity 值。

### GetParentValueSql() — 查詢主表欄位值

產生一條 SELECT SQL，從主表查詢指定欄位值。供 InfoTransaction 交易回寫時使用。

```sql
SELECT {field} FROM {masterTable} WHERE {masterColumn} = {detailRow[detailColumn]}
```

## 關聯架構

```
masterCommand（主表 InfoCommand）
    │
    ├── InfoDataSource（關聯定義）
    │   ├── masterColumns: "ORDER_NO"
    │   └── detailColumns: "ORDER_NO"
    │
    └── detailCommand（明細 InfoCommand）

查詢流程：
  選取主表行 → parentRow = { ORDER_NO: "ORD001" }
  → GetDetailSql() → SELECT * FROM OrderDetail WHERE ORDER_NO = 'ORD001'

刪除流程：
  刪除主表行 → parentRow = { ORDER_NO: "ORD001" }
  → GetDeleteDetailSqls()
    → DELETE FROM SubDetail WHERE ORDER_NO = 'ORD001'  ← 先刪子明細
    → DELETE FROM OrderDetail WHERE ORDER_NO = 'ORD001' ← 再刪明細
  → DELETE FROM Orders WHERE ORDER_NO = 'ORD001'        ← 最後刪主表

新增流程：
  INSERT 主表（Identity ID=100）
  → GetParentValues() → { ORDER_NO: "100" }
  → INSERT 明細 → ORDER_NO 自動填入 "100"
```

## 備註

- MasterColumns 和 DetailColumns 必須數量相同且位置對應（第 1 個對第 1 個）。
- 刪除連動是遞迴的：若明細表本身也有下層明細（透過另一個 InfoDataSource），會先刪除最底層再往上刪。
- 刪除連動中的錯誤訊息會被設為空字串（`Options.SqlErrorMessage[sql] = ""`），避免因明細不存在而報錯。
- `GetParentValues()` 也會遞迴向上查：若主表也是別人的明細，會沿著關聯鏈取到最頂層的值。
- SystemTable.json 中的 `idsMenuUser`、`idsMenuGroup`、`idsSysParas`、`idsScheduleLog` 等都是 InfoDataSource 的實例。
