# DetailFlowActivity（明細流程活動）

> 設計端：`EEPFlowTools/Activities/DetailFlowActivity.cs`（42 行）
> 引擎端：`EEP.Flow/Activities/DetailFlowActivity.cs`（470 行）

## 用途

明細流程活動，用於根據明細資料表的每一筆記錄**各自啟動一個子流程**進行會簽。例如：一張採購單有 3 筆明細，每筆明細分別發起獨立的簽核流程，全部完成後主流程才繼續。繼承自 `CompositedActivity`（複合活動），內部管理多個子活動（`ChildActivities`）。工具箱中對應為 **CountersignActivity**。

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| Title | string | （繼承自 Activity） | 活動標題（顯示名稱） |
| FlowName | string | `MenuEditor("netflow")` | 子流程名稱（明細要跑的流程） |
| FormName | string | `MenuEditor("bootstrap")` | 網頁表單名稱（明細的表單頁面） |
| TabTitle | string | — | 頁簽標題 |
| Keys | `List<FlowKey>` | `CollectionEditor("netflow", "detailFlowActivity", "flowKey", "field")` | 流程主鍵集合 |
| DetailTableName | string | `TableEditor` | 明細資料表名稱 |
| MasterColumns | string | — | 主表關聯欄位（逗號分隔） |
| DetailColumns | string | `ColumnEditor("", true)` | 明細表關聯欄位（逗號分隔） |
| Expression | string | — | 篩選條件表達式 |

### FlowKey 子類別

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| Title | string | — | 主鍵標題（顯示用） |
| Field | string | `ColumnEditor` | 主鍵欄位名稱 |

## 引擎執行邏輯

### Start() 主流程（啟動明細子流程）

```
1. 檢查流程狀態（非 Return / Retake 才執行）
2. 設定 isInitialized = true，清空 ChildActivities
3. 查詢明細資料表 GetDetailTable()
   ├─ 根據 MasterColumns ↔ DetailColumns 對應關係，從主表取值作為查詢條件
   ├─ 呼叫 SelectUserTable(DetailTableName, parameters) 取得明細資料
   └─ 若有 Expression，逐筆以 FlowExpression.JudgeExpression() 過濾
4. 對每筆明細記錄：
   ├─ 轉為 JSON → 組合 Parameter（含 WEBFORM_NAME、tabTitle、FORM_KEYS 等）
   ├─ 唯讀模式 → 建立空的 DetailResultActivity
   └─ 一般模式 → 呼叫 StartDetail() 啟動子流程 → 建立 DetailResultActivity（含結果 DataSet）
5. 若有子活動 → InsertTodo()（寫入待辦）
6. 呼叫 base.Start() 繼續複合活動邏輯
```

### GetDetailTable() 明細查詢

```
1. 解析 MasterColumns / DetailColumns（逗號分隔）建立對應關係
2. 從流程參數取得主表欄位值 → 作為查詢明細表的條件
3. 呼叫 SelectUserTable() 查詢明細資料
4. 若有 Expression → 逐筆以 FlowExpression 判斷，不符合條件的排除
5. 回傳篩選後的 DataTable
```

### GetParameter() 參數組合

將明細記錄的 JSON 加入以下固定欄位：

| 欄位 | 值 |
|------|-----|
| `WEBFORM_NAME` | FormName 屬性 |
| `tabTitle` | TabTitle 屬性 |
| `FORM_KEYS` | Keys 的 Field 以分號串接 |
| `FORM_PRESENTATION` | 各 Key 的 `Field='值'` 以分號串接 |
| `FORM_PRESENTATION_CT` | 各 Key 的 `Title:值` 以分號串接（Title 空時用 Field） |

### GetDetailRows() 取得明細狀態

根據 `IsInitialized` 狀態決定資料來源：

| 狀態 | 來源 | 說明 |
|------|------|------|
| 已初始化 | ChildActivities | 從子活動取得 InstanceID/Parameter，查歷程判斷狀態（Reject/End/Submit） |
| 未初始化 | 明細資料表 | 從 GetDetailTable() 取得，狀態為 Start |

### 輔助類別

#### DetailResultActivity

子流程的結果活動，代表一筆明細的簽核狀態。

| 特性 | 值 |
|------|-----|
| 繼承自 | `Activity` |
| `IsWaiting` | `true`（等待子流程完成） |
| `SupportReturn / Plus / Agent / Reject / Edit / Print` | 全部 `false` |
| `DurationDays` | `0` |

關鍵行為：
- `Start()` → 狀態設為 Waiting，記錄開始時間
- `Execute()` → 狀態設為 Executed；若父活動（DetailFlowActivity）已全部完成 → 呼叫 `DeleteTodo()` 移除待辦
- `LogDetail(status)` → 寫入歷程（帶狀態參數）

#### DetailPreviewActivity

預覽用活動，顯示明細子流程的執行進度。

| 特性 | 值 |
|------|-----|
| `Executed` | 已完成數量 |
| `Total` | 總數量 |
| `ToObject()` | 輸出含 `Executed/Total` 進度文字 |

### 關鍵方法一覽

| 方法 | 說明 |
|------|------|
| `Start()` | 啟動所有明細子流程 |
| `Reset()` | 重設 isInitialized 為 false 並呼叫 base.Reset() |
| `GetDetailTable()` | 查詢並篩選明細資料 |
| `GetDetailRows()` | 取得明細子流程狀態清單 |
| `GetParameter()` | 組合明細記錄的流程參數 JSON |
| `StartDetail()` | 呼叫 `Instance.Service.StartDetail()` 啟動子流程 |
| `InsertTodo()` | 透過 `FlowTodoHelper.Insert()` 寫入待辦 |
| `DeleteTodo()` | 透過 `FlowTodoHelper.Delete()` 移除待辦 |
| `CreatePreviewActivity()` | 建立預覽活動（含已完成/總數） |

## 備註

- 工具箱顏色為淺灰色（#eee），工具箱顯示名稱為 **CountersignActivity**
- 繼承自 `CompositedActivity`（而非 `ContinuedActivity`），具備管理子活動的能力
- 明細子流程透過 `Instance.Service.StartDetail()` 啟動，每筆明細各自獨立運作
- 子流程完成時由 `DetailResultActivity.Execute()` 檢查父活動狀態，全部完成才移除待辦
- `IsPreviewStart` 標記用於唯讀模式（預覽），此時不實際啟動子流程
- `Expression` 篩選使用 `FlowExpression.JudgeExpression()`，可依明細欄位值決定是否需要跑子流程
- MasterColumns 與 DetailColumns 必須一一對應，數量需一致
