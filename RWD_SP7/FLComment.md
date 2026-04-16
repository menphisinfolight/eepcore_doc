# FLComment

> C# 端：`EEPRWDTools.Core/Controls/FLComment.cs` — 16 行
> 前端：`bootstrap.infolight.nodeflow.js` — `$.fn.flcomment`（line 445–500）
> 繼承：`RWDControl` → `Component`

## 用途

**流程簽核歷程元件**（FL Comment）。

FLComment 在 DataForm 中顯示該筆資料的**完整簽核歷程表格**，包含每個關卡的簽核者、狀態、意見和時間。C# 端只渲染一個空 `<div>`，前端 JS 動態產生 flowgrid 表格並從 FLOWHISTORY 載入資料。

## 設計介面

**無任何可設定屬性**。從工具箱拖入 DataForm 中即可。

```json
{
  "type": "flcomment",
  "id": "FLComment1"
}
```

## 顯示欄位

FLComment 固定顯示以下 5 個欄位（**不可自訂**）：

| 欄位 | 來源欄位 | Formatter | 說明 |
|------|---------|-----------|------|
| **關卡名稱** | `ActivityText` | `formatterActivityText` | 活動名稱，自動翻譯後綴（Plus→加簽、Notify→通知、Reject→作廢等） |
| **簽核人** | `UserName` | — | 簽核者姓名，代理簽核會帶 `(*)` 標記 |
| **狀態** | `Status` | `formatterStatus` | 數值轉文字（見下方對照表） |
| **意見** | `Remark` | — | 簽核意見/備註 |
| **日期** | `Datetime` | — | 簽核時間 |

### Status 狀態對照表

| 值 | 顯示文字 | 說明 |
|----|---------|------|
| 0 | 送出（submit）/ 審核（approve） | 0 且 ActivityID='StartActivity' 顯示「送出」，否則「審核」 |
| 1 | 審核（approve） | 正常簽核 |
| 2 | 退回（back） | 退回上一步 |
| 3 | 取回（retake） | 送出後反悔取回 |
| 5 | 加簽（plus） | 加簽給其他人 |
| 6 | 轉簽（transfer） | 轉簽給其他人 |
| 7 | 作廢（reject） | 流程作廢 |
| 8 | 起單（startFlow） | 流程啟動 |
| 9 | 結案（finished） | 流程完成 |
| 10 | 會簽（multiapprove） | 多人會簽 |
| 11 | 通知（notify） | 知會通知 |

### ActivityText 格式化

簽核歷程中的活動名稱會自動翻譯特殊後綴：

| 原始值 | 顯示結果 |
|--------|---------|
| `主管審核` | 主管審核 |
| `主管審核_Plus` | 主管審核_加簽 |
| `主管審核_Notify` | 主管審核_通知 |
| `主管審核_Reject` | 主管審核_作廢 |
| `主管審核_AgentApprove` | 主管審核_代理審核 |
| `主管審核_AgentReturn` | 主管審核_代理退回 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.nodeflow.js` line 445–500
> jQuery Plugin：`$.fn.flcomment`（透過 `$.createFlowObj` 建立）

### 初始化流程

```text
1. 在 <div class="bootstrap-flcomment"> 內動態產生 flowgrid 表格
2. 檢查 sessionStorage 中是否有流程資料（InstanceID）
   → 有 → 直接載入歷程（flowgrid.load）
   → 沒有 → 隱藏自己，等待 DataForm 開啟時觸發
3. 監聽 DataForm 的 onFlowLoad 事件
   → 從 FlowFlag 欄位解析 InstanceID（格式："狀態碼:InstanceID"）
   → 有 InstanceID → 載入歷程並顯示
   → 沒有 → 保持隱藏
```

### 資料來源

FLComment 透過 `flowgrid` 元件以 `mode: 'queryFlow', type: 'Detail'` 向後端查詢，後端從 **FLOWHISTORY** 表撈取該 InstanceID 的所有簽核歷程。

```text
前端 → flowgrid.load() 
     → POST /flow (mode=queryFlow, type=Detail, InstanceID=xxx)
     → FlowHelper → API.QueryHistory(instanceID)
     → FLOWHISTORY 表查詢
     → 回傳簽核歷程資料
```

### FlowFlag 解析

FLComment 需要資料表中有一個 `FlowFlag` 欄位（或 `FLOWFLAG`）來取得 InstanceID：

```text
FlowFlag 格式："{狀態碼}:{InstanceID}"
例如："N:a1b2c3d4-e5f6-7890-abcd-ef1234567890"

狀態碼：
  P = Prepare（預啟動）
  N = Normal（進行中）
  Z = End（結案）
  X = Reject（作廢）
```

## ⚠️ 不可自訂

| 項目 | 可否自訂 | 說明 |
|------|:-------:|------|
| 顯示欄位 | ❌ | 固定 5 欄（關卡/簽核人/狀態/意見/日期），寫死在 JS 中 |
| 欄位順序 | ❌ | 固定順序 |
| 欄位寬度 | ❌ | 固定 80px |
| 分頁設定 | ⚠️ 半固定 | 固定 pageSize=10，有分頁功能 |
| 資料來源 | ❌ | 固定從 FLOWHISTORY 查詢 |
| 樣式 | ⚠️ 可透過 CSS 覆蓋 | 使用 `#dgHistoryRemark` 選擇器 |

如果需要自訂簽核歷程的顯示，建議不要使用 FLComment，改為自行建立 DataGrid 搭配 `onBeforeLoad` 事件查詢 FLOWHISTORY。

## 備註

- FLComment 預設**隱藏**，只有在 DataForm 載入到有 FlowFlag 的資料時才顯示。
- 如果資料表沒有 FlowFlag 欄位，FLComment 永遠不會顯示。
- 表格 ID 固定為 `dgHistoryRemark`，同一頁面放多個 FLComment 會有 ID 衝突。
- `$.createFlowObj` 和 `$.createObj` 類似，但是流程專用的元件建立方法。
