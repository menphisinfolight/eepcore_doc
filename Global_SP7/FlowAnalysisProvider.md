# FlowAnalysisProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/FlowAnalysisProvider.cs` |
| 行數 | 212 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

流程分析 Provider，針對簽核流程歷史資料提供統計分析，包含流程總量分析、流程時效分析及流程瓶頸分析。

## type 分派

注意此 Provider 使用 `param["type"]` 而非 `param["mode"]` 進行分派。

| type | 說明 |
|------|------|
| `getFlowText` | 取得不重複的 FlowID 清單 |
| `totalFlowAnalysis` | 流程總量分析（待辦 vs 已結案數量） |
| `timeEffectivenessAnalysisOfProcess` | 流程時效分析（平均完成天數） |
| `processBottleneckAnalysis` | 流程瓶頸分析（各節點平均停留天數） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `TotalFlowAnalysis(...)` | 分別查 FLOWTODO 和 FLOWHISTORY 的 Instance 數，合併回傳 |
| `TimeEffectivenessAnalysisOfProcess(...)` | 計算已結案流程的平均耗時（天），支援 Informix/MSSQL/Oracle |
| `ProcessBottleneckAnalysis(...)` | 計算各活動節點的平均停留天數 |
| `CreateCommandText(...)` | 動態組裝 SQL（WHERE + GROUP BY），處理日期與 FlowID 篩選 |

## 備註

- 支援多種資料庫（Informix、MSSQL、Oracle），各自有不同的日期差異計算方式
- MSSQL 使用 CTE（WITH FlowTime AS）來連結開始與結束記錄
- 資料表名稱依資料庫類型有大小寫差異（FlowHistory vs FLOWHISTORY）
