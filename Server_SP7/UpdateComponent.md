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

| 事件 | 簽名 | 回傳 | 說明 |
|------|------|------|------|
| **onBeforeApply** | `void fn(sender, rows, sqls)` | — | 整批資料處理前（rows 含 inserted/updated/deleted） |
| **onBeforeInsert** | `bool fn(sender, row, sqls)` | false=跳過 | 單筆 INSERT 前 |
| **onBeforeUpdate** | `bool fn(sender, row, sqls)` | false=跳過 | 單筆 UPDATE 前 |
| **onBeforeDelete** | `bool fn(sender, row, sqls)` | false=跳過 | 單筆 DELETE 前 |
| **onAfterInsert** | `void fn(sender, row, sqls)` | — | 單筆 INSERT 後（可追加 SQL） |
| **onAfterUpdate** | `void fn(sender, row, sqls)` | — | 單筆 UPDATE 後 |
| **onAfterDelete** | `void fn(sender, row, sqls)` | — | 單筆 DELETE 後 |
| **onAfterApply** | `void fn(sender, rows, sqls)` | — | 整批 SQL 產生完成後（Transaction 前） |
| **onAfterApplied** | `void fn(sender, rows, sqls)` | — | Transaction Commit 後（由 DataModule 觸發） |

### Before 事件的回傳值

- `return true`（預設）：繼續執行預設 SQL
- `return false`：跳過預設 SQL（通常在事件中自行組 SQL 加入 sqls 列表）

常見用法：

```csharp
// 自行組 SQL，跳過預設 INSERT
public bool ucUserCard_onBeforeInsert(object sender, dynamic row, List<string> sqls)
{
    sqls.Add($"INSERT INTO USERCARDS (USERID, CARDID) VALUES ('{row.USERID}','{row.CARDID}')");
    return false; // 跳過預設
}
```

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
