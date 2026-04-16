# FlowProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/FlowProvider.cs` |
| 行數 | 254 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `AccountProvider` |

## 用途

簽核流程操作 Provider，提供流程列表查詢、流程方法呼叫、流程定義取得等功能。同一檔案包含 `EEPNetFlowProvider`，提供 EEPNetFlow 的相似操作。

## FlowProvider mode 分派

| mode | 說明 |
|------|------|
| `getFlows` | 取得流程清單（無 type 參數時讀 XML 檔名，有 type 時走 FlowHelper 查詢） |
| `callFlowMethod` | 呼叫流程方法（委託 FlowHelper） |
| `queryFlow` | 查詢流程（委託 FlowHelper） |
| `queryData` | 查詢流程資料（委託 FlowHelper） |
| `getDefination` | 取得流程定義（活動類型 + 屬性清單），透過 `Flow.QueryFlow` + 反射取活動型別 |

## EEPNetFlowProvider mode 分派

| mode | 說明 |
|------|------|
| `getFlows` | 取得 EEPNetFlow 流程清單 |
| `callFlowMethod` | 呼叫 EEPNetFlow 方法 |
| `queryFlow` | 查詢 EEPNetFlow 流程 |
| `queryData` | 查詢流程資料 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetDefination(param)` | 反射取得所有 Activity 子型別及其屬性，回傳 controls + types |
| `GetClientParam()` | 組裝流程 API 所需參數（conn_str、user、database 等） |
| `LogonWithKey()` | SSO 登入後取得流程相關資訊，回傳 email 參數 |

## 備註

- `FlowProvider` 繼承 `AccountProvider`，具備 SSO 登入能力
- `EEPNetFlowProvider` 額外提供 `UploadFile` / `DownloadFile` 方法
- 流程定義檔位於 `design/netflow/{solution}/` 下的 `.xml` 檔
