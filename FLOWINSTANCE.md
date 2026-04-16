# FLOWINSTANCE

## 用途

**流程實例表**（Flow Instance）。

FLOWINSTANCE 是流程引擎的核心表，儲存每個流程實例的完整狀態（二進位序列化的 Instance 物件）與參數。每筆記錄代表一次流程的執行，從啟動到結案的整個生命週期都依賴此表。

### 使用場景

| 場景 | 說明 |
|------|------|
| **流程啟動** | API.Start() 建立新的 Instance 物件後，序列化寫入 State 欄位 |
| **簽核執行** | Instance.Submit() 每次儲存時先 DELETE 再 INSERT（整筆覆蓋） |
| **流程載入** | API.LoadInstance() 從 State 欄位反序列化還原 Instance 物件 |
| **流程結案/刪除** | Instance.Delete() 同時刪除 FLOWINSTANCE、FLOWTODO、FLOWHISTORY |

### 關聯表

```
FLOWINSTANCE ──(InstanceID)──> FLOWTODO      （待辦事項）
             ──(InstanceID)──> FLOWHISTORY   （簽核歷史）
             ──(InstanceID)──> FLOWNOTIFY    （通知）
             ──(InstanceID)──> FLOWCOMMENT   （簽核意見）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowInstanceHelper.cs` | 資料存取：Get / Insert / Delete / Update |
| `EEP.Flow/Instance.cs` | 流程實例核心邏輯：Save() / Delete() / Submit() |
| `EEP.Flow/API.cs` | 流程 API：LoadInstance() / Start() / Prepare() |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.flowdesigner.js` | 前端流程設計器：判斷 InstanceID 是否存在 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **InstanceID** | `nvarchar(50)` PK | NOT NULL | 流程實例唯一識別碼 |
| **State** | `image` | NOT NULL | 流程實例的完整狀態（BinaryFormatter 序列化的 Instance 物件） |
| **Datetime** | `datetime` | NOT NULL | 最後更新時間（每次 LoadInstance 時更新） |
| **Parameter** | `nvarchar(2048)` | NULL | 流程參數（JSON 格式） |
| **ProjectID** | `nvarchar(50)` | NULL | 所屬解決方案編號 |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 |
|------|-----------|-------|--------|-----|
| State | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |
| Datetime | `datetime` | `datetime` | `date` | `TIMESTAMP` |
| Parameter | `nvarchar(2048)` | `nvarchar(2048)` | `clob` | `NVARCHAR(2048)` |

---

## 主鍵

```
PRIMARY KEY (InstanceID)
```

---

## 資料生命週期

```
建立：API.Start() → NewInstance() → FlowInstanceHelper.Insert()
讀取：API.LoadInstance() → FlowInstanceHelper.Update(更新Datetime) → FlowInstanceHelper.Get(反序列化)
儲存：Instance.Save() → FlowInstanceHelper.Delete() → FlowInstanceHelper.Insert()（先刪後寫）
刪除：Instance.Delete() → 同時刪除 FLOWINSTANCE / FLOWTODO / FLOWHISTORY
```

---

## State 欄位內容

State 欄位以 BinaryFormatter 序列化整個 Instance 物件，反序列化後包含：

| 屬性 | 說明 |
|------|------|
| InstanceID | 流程實例編號 |
| FlowID | 流程定義編號 |
| Parameter | 流程參數（JSON） |
| Remark | 簽核意見 |
| Defination | 流程定義文件 |
| TextSuffix | 流程名稱後綴 |
| Database | 資料庫 |
| ProjectID | 解決方案編號 |
| MasterInstanceID | 主流程編號（子流程時使用） |
| EndStatus | 結案時的狀態 |
| Status | 當前流程狀態（InstanceStatus 列舉） |
| RootActivity | 根活動 |
| CurrentActivity | 當前活動 |

---

## 備註

- Save() 採用「先 DELETE 再 INSERT」策略，而非 UPDATE State 欄位，因為 State 為整個物件的序列化結果，每次都需完整覆寫。
- LoadInstance() 會先執行 `UPDATE FlowInstance SET Datetime = @now WHERE InstanceID = @id`，用於記錄最後存取時間。
- 反序列化使用自訂的 `FlowInstanceBinder`，確保跨版本相容性。
- LoadInstance() 會檢查流程狀態，若為 Reject 或 End 狀態則拋出異常。
- 前端流程設計器透過判斷 `InstanceID` 是否存在來區分新建或編輯流程。

---
