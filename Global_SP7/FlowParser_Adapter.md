# FlowParser (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/FlowParser.cs` |
| 行數 | 1302 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

將 AI 產出的流程 JSON（含節點、類型、對象、條件、下一步等中文欄位）解析並轉換為 EEP NetFlow 設計器的圖形化 JSON 定義（含 activity、segment、line 物件及座標定位）。

## 主要類別

### FlowParser

| 方法/屬性 | 說明 |
|-----------|------|
| `FlowParser(clientInfo, json)` | 解析 JSON 並建立 Activities / Segments / Lines |
| `ToJSON()` | 輸出完整流程 JSON（segment + activity + line + flow） |
| `Activities` | 所有活動節點（含位置資訊） |
| `Segments` | 泳道分段資訊 |
| `Lines` | 連線資訊 |

### 節點類型常數

| 常數 | 對應中文 | 活動型別 |
|------|---------|---------|
| `START_VALUE` | 開始 | `StartActivity` |
| `END_VALUE` | 結束 | `EndActivity` |
| `STAND_VALUE` | 作業 | `StandActivity` |
| `APPROVE_VALUE` | 審核 | `StandActivity` |
| `NOTIFY_VALUE` | 通知 | `NotifyActivity` |
| `IFELSE_VALUE` | 條件 | `IfElseActivity` |
| `PARALLEL_VALUE` | 平行 | `ParallelActivity` |
| `PROCEDURE_VALUE` | 程序 | `ProcedureActivity` |
| `VALIDATE_VALUE` | 稽核 | `ValidateActivity` |
| `SUBFLOW_VALUE` | 子流程 | `SubFlowActivity` |

### ActivityInfo

每個節點的完整資訊，包含：
- 座標計算（Left / Top / Size）
- 容器節點（IfElse / Parallel）的子節點佈局
- `SendTo` 解析：`user:xxx` / `role:xxx` / `refrole:xxx` / `Applicant` / `Manager`
- 使用者/角色名稱查 DB 對應 ID

### SegmentInfo / LineInfo / FlowInfo

泳道（segment）、連線（line）及流程根節點（flow）的 JSON 輸出。

## 解析流程

1. 讀取 JSON 的「流程名稱」和「節點」陣列
2. `ProcessItems`：移除無效分支、自動串接未連結節點
3. `ParseNext`：遞迴排序節點，自動建立 IfElse / Parallel 容器
4. 計算泳道寬度與座標定位
5. `ToJSON` 輸出

## 備註

- 泳道顏色預設 4 色循環：`#aaffaa` / `#ffffaa` / `#ffd4aa` / `#aaffff`
- 活動間距 `ACTIVITY_MARGIN = 30`
- 查詢 GROUPS / USERS 資料表取得群組名稱→ID 對應
