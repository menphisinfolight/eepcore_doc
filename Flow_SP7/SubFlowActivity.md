# SubFlowActivity（子流程活動）

> 設計端：`EEPFlowTools/Activities/SubFlowActivity.cs`（14 行）
> 引擎端：`EEP.Flow/Activities/SubFlowActivity.cs`（92 行）

## 用途

子流程活動，將另一個流程定義嵌入作為目前流程的一部分。支援兩種方式：指定流程名稱（靜態）或從欄位值取得流程定義（動態）。

## 繼承

```
Activity → CompositedActivity → SequenceActivity → SubFlowActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `FlowName` | string | `MenuEditor("netflow")` | 子流程名稱（從流程選單選取） |
| `FlowField` | string | — | 動態流程欄位名稱 |

## 引擎執行邏輯

### Start()

兩種模式擇一：

**模式一：FlowName（靜態子流程）**
1. 從 `Instance.Service.FlowFilePath` 讀取 `{FlowName}.xml`
2. 反序列化為 `RootActivity`
3. 將子流程中所有活動加入本活動的 `ChildActivities`
4. 重新命名子活動 ID 為 `{本活動ID}_{原ID}`，避免 ID 衝突

**模式二：FlowField（動態子流程）**
1. 從 `Instance.GetParameterValue(FlowField)` 取得 JSON 字串
2. 透過 `Instance.Service.Parser.ToActivities(json)` 解析為活動清單
3. 同樣加入 ChildActivities 並重新命名 ID

### SetSubID()

遞迴處理巢狀的 `CompositedActivity`，確保所有層級的子活動 ID 都加上前綴。

### 狀態判定

繼承 `SequenceActivity`，子活動依序執行，全部完成後本活動 Executed。

## 備註

- 子流程是「嵌入式」而非「獨立實例」，不會產生新的流程單據
- `FlowName` 和 `FlowField` 為互斥設定，FlowName 優先
- 動態模式適用於流程結構需依資料決定的場景
