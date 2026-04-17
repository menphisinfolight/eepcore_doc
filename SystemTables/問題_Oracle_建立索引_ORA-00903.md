# 問題：Oracle 從設計介面建立索引失敗 ORA-00903

> **回報日期**：2026-04-17
> **回報客戶**：昇銳電子 / 邱顯宗
> **影響版本**：EEP SP7
> **影響資料庫**：Oracle（11g 已確認；其他 Oracle 版本也受影響）
> **相關討論**：https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482587

## 問題描述

在 EEP 設計介面的「資料庫工具 → 索引管理」中建立索引時，Oracle 資料庫回傳錯誤：

```
ORA-00903: 表名無效
```

實際操作：對 `EMPLOYEE` 表的 `CNAME` 欄位建立索引 `IDX_EMPLOYEE_CNAME`。

## 根本原因

`EEPBase.Core/Utility/DatabaseHelper.cs` 中 **`OracleHelper.GetCreateIndex`**（L2835-2838）產生 SQL 時，**表名用錯引號**：

```csharp
public override string GetCreateIndex(string tableName, string indexName, string columnStr)
{
    return "create index " + indexName + " on \'" + tableName + "\' (" + columnStr + ")";
    //                                          ^^                  ^^
    //                                    單引號 '，應為雙引號 " 或無引號
}
```

產生的 SQL：

```sql
CREATE INDEX IDX_EMPLOYEE_CNAME on 'EMPLOYEE' (CNAME)
```

Oracle 識別字規則：

| 符號 | 意義 |
|------|------|
| `"TABLE"` | 識別字（區分大小寫的表名） |
| `'TABLE'` | **字串字面值** |
| `TABLE` | 識別字（Oracle 自動轉大寫） |

由於用了單引號，Oracle 將 `'EMPLOYEE'` 視為**字串**（不是表名），語法不合 `CREATE INDEX ... ON {表名}` 語法 → 拋 `ORA-00903`。

## 5 個資料庫實作比對

| 資料庫 | 行號 | 產生 SQL | 狀態 |
|--------|------|----------|------|
| SQL Server | L2108 | `create index XXX on TABLE (col)` | ✅ 正常 |
| **Oracle** | **L2835** | `create index XXX on ` **`'TABLE'`** ` (col)` | ❌ **Bug** |
| MySQL | L3368 | `` create index XXX on `TABLE` (col) `` | ✅ 正常 |
| DB2 | L4192 | `CREATE INDEX XXX ON {MarkTable} (col)` | ✅ 正常 |
| Informix | L5053 | `CREATE INDEX XXX ON {MarkTable} (col)` | ✅ 正常 |

只有 Oracle 版本有此 typo。其他四個資料庫在設計介面建索引皆能正常建立。

## 呼叫鏈

```
前端（設計介面 → 索引管理 → 建立）
  ↓ POST /index/create, { table, index, strs }
EEPGlobal.Core/Provider/IndexProvider.CreateIndex()
  ↓ 呼叫 DbHelper
DatabaseHelper.GetCreateIndex()  ← OracleHelper 版本有 bug
  ↓
DatabaseHelper.ExecuteNonQuery()  ← SQL 丟給 Oracle 執行
  ↓
Oracle → ORA-00903
```

## 修法

### 方法 1：最小修改（`\'` 改 `\"`）

```csharp
public override string GetCreateIndex(string tableName, string indexName, string columnStr)
{
    return "create index " + indexName + " on \"" + tableName.ToUpper() + "\" (" + columnStr + ")";
}
```

注意：Oracle 在 `"..."` 中會**區分大小寫**，若表名在資料庫中是大寫存儲（預設），傳入的 `tableName` 須轉大寫。

### 方法 2：與 DB2/Informix 一致，用 `MarkTable`

```csharp
public override string GetCreateIndex(string tableName, string indexName, string columnStr)
{
    return "CREATE INDEX " + indexName + " ON " + MarkTable(tableName) + " (" + columnStr + ")";
}
```

這樣處理引號/大小寫的邏輯交給 `MarkTable()`，與其他資料庫的程式碼風格一致。

### 方法 3：簡單做，不加引號

```csharp
public override string GetCreateIndex(string tableName, string indexName, string columnStr)
{
    return "create index " + indexName + " on " + tableName + " (" + columnStr + ")";
}
```

Oracle 對不加引號的識別字會自動處理（轉大寫），若系統中表名都是標準大寫名稱，這樣最簡單。但不支援含特殊字元或保留字的表名。

## 對照廠商回覆

內部人員 Roland 於 2026-04-17 回覆：

> #1：如要建立 ORACLE 資料庫索引，請直接透過 ORACLE 資料庫工具進行相關設定。關於此報錯，我們內部研究看看，會於下個版本 SP8 中修正。
>
> #2：補充，目前此功能我測試，雖然有介面，但沒有實際在資料庫設定索引。因此還請同樣參考 1 樓，先透過資料庫工具設定。

**Roland 的描述不完全正確**：

- `IndexProvider.CreateIndex()` **有實際呼叫** `dbHelper.ExecuteNonQuery(sql)`，真的會送 SQL 到資料庫執行
- 只是 Oracle 版本產生的 SQL 被資料庫拒絕（ORA-00903），因此**看起來像沒設定**
- 在 **SQL Server / MySQL / DB2 / Informix** 此功能實際可用，能成功建立索引

## 暫時解法（客戶端）

在 SP8 修正釋出前，Oracle 使用者：

1. 使用 Oracle 原生工具（SQL Developer、PL/SQL Developer、sqlplus）建立索引
2. 或直接執行 SQL：
   ```sql
   CREATE INDEX IDX_EMPLOYEE_CNAME ON EMPLOYEE (CNAME);
   ```

## 類似風險

`DropIndex` 在 Oracle 版本（L2840-2843）沒有此問題，因為 Oracle `DROP INDEX` 不需要表名：

```csharp
public override string GetDropIndex(string tableName, string indexName)
{
    return "drop index " + indexName;
}
```

但在 SQL Server（L2113）用的是 `tableName + "." + indexName` 語法，Oracle 不需要 — 這部分目前是對的。

## 備註

- 此 bug 存在於 SP7 所有小版本
- 建議同步檢查其他 `GetCreateXxx` 系列方法是否有類似引號錯誤
- 官方 CHM 文件未提及此限制，使用者可能會困惑「為什麼 Oracle 建索引一直失敗」
