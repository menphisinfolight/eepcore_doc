# ValidateActivity（驗證活動）

> 設計端：`EEPFlowTools/Activities/ValidateActivity.cs`（14 行）
> 引擎端：`EEP.Flow/Activities/ValidateActivity.cs`（70 行）

## 用途

驗證活動，在流程推進時檢查條件是否成立。若驗證失敗，產生 `ExceptionActivity` 中斷流程並顯示錯誤訊息。

## 繼承

```
Activity → ContinuedActivity → ValidateActivity
```

## 設計介面屬性

| 屬性 | 型別 | 編輯器 | 說明 |
|------|------|--------|------|
| `Expression` | string | — | 條件運算式 |
| `Function` | string | `ServerMethodEditor` | 伺服器方法名稱 |
| `Message` | string | — | 驗證失敗時的錯誤訊息 |

## 引擎執行邏輯

### Invoke()

1. 若有 `Expression`：呼叫 `Instance.JudgeExpression(Expression)` 判定
2. 若有 `Function`：呼叫 `Instance.Invoke(Function)`
   - 回傳 "True" → 驗證通過
   - 回傳非 "True" 且非 "False" → 將回傳值作為 `Message`（動態錯誤訊息）
3. 驗證失敗 → 建立 `ExceptionActivity` 並回傳，阻止流程繼續

### 流程行為（繼承自 ContinuedActivity）

- `IsWaiting = false`：不需等待人工操作
- `Start()` 呼叫 `Invoke()` 後，若回傳包含 `SystemActivity`（即 ExceptionActivity），則**不會**標記為 Executed，流程中斷

## 備註

- 驗證通過時回傳空清單，流程自動繼續到下一個活動
- `Function` 可回傳自訂錯誤訊息（非 True/False 字串），比固定 `Message` 更靈活
