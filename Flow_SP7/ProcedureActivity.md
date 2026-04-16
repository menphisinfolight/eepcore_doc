# ProcedureActivity（程序活動）

> 設計端：`EEPFlowTools/Activities/ProcedureActivity.cs`（11 行）
> 引擎端：`EEP.Flow/Activities/ProcedureActivity.cs`（51 行）

## 用途

程序活動，在流程推進時呼叫伺服器自訂方法。不需人工簽核，執行完畢自動繼續。常用於流程中的資料處理、狀態更新等自動化邏輯。

## 繼承

```
Activity → ContinuedActivity → ProcedureActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `Function` | string | `ServerMethodEditor(true)` | 伺服器方法名稱（必填） |

## 引擎執行邏輯

### Invoke()

1. 檢查 `Instance.Service.ReadOnly`，唯讀模式下不執行
2. 呼叫 `Instance.Invoke(Function)` 執行方法
3. 呼叫 `Instance.RefreshParameter()` 重新整理參數
4. 回傳空清單（不產生新活動）

### Reset()

若活動已為 Executed 狀態，**退回時會再次呼叫 `Invoke()`**（補償/回滾邏輯），然後呼叫 `base.Reset()`。

### 引擎端額外屬性

| 屬性 | 型別 | 預設值 | 說明 |
|------|------|--------|------|
| `PreviewVisible` | bool | false | 是否在流程預覽中顯示 |

## 備註

- 退回時的 Reset 會重新執行方法，設計方法時需考慮冪等性或反向操作
- `PreviewVisible` 預設為 false，表示程序活動通常不在流程預覽中顯示
