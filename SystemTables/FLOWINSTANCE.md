# FLOWINSTANCE

> ⚠️ **實際表名是 `FlowInstance`（CamelCase），不是全大寫 `FLOWINSTANCE`**。`CREATE TABLE FlowInstance(...)` — SQL Server / MySQL 大小寫不敏感還好，**Oracle、DB2、Informix 下手寫 SQL 要用 `"FlowInstance"` 加雙引號**，用大寫 `FLOWINSTANCE` 查不到。此文件檔名沿用舊版大寫只為了側邊欄對齊。

## 用途

**流程實例表**（Flow Instance）— SP7 唯一有效的流程狀態儲存表。

每次啟動流程，引擎會把**完整的 `Instance` 物件**（包含流程定義 XML、當前活動、等待活動、參數、簽核意見等）用 `BinaryFormatter` 序列化成 binary blob 寫進 `State` 欄位。因此「當下的流程圖」—— 不只是執行到哪個節點，連**流程設計檔本身**也一併快照進去（`Instance.Defination` 屬性）。

這帶來**版本隔離效果**：即使 `SYS_FLDEFINITION` 的設計檔之後被修改，已啟動的實例仍照舊版流程定義繼續跑，不會受影響。

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

五個 DB CREATE TABLE 都完整。型別差異：`State` 用 BLOB/image、`Datetime` 各 DB 日期型別不同、`Parameter` 各 DB 為 nvarchar/clob/LVARCHAR。

👉 升級 ALTER 支援矩陣、手動補欄位 SQL：**問題_SP7_跨資料庫欄位差異盤點**

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
- **舊版 `SYS_FLINSTANCESTATE` 已棄用**：SP7 所有 C# / 前端程式碼都不引用，只有建表腳本殘留，千萬別寫新程式去讀那張表。

## 🔴 技術債：`BinaryFormatter` 已被 .NET 淘汰

`FlowInstanceHelper.cs` 的 `Insert()` / `Get()` 使用 `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter`：

| .NET 版本 | 狀態 |
|-----------|------|
| .NET Framework | 可用 |
| .NET 5 / 6 / 7 | 標記 obsolete，需在 csproj 開啟 `<EnableUnsafeBinaryFormatterSerialization>true</EnableUnsafeBinaryFormatterSerialization>` |
| **.NET 8+** | **預設已移除**，連 opt-in flag 都不再有效 |

升級到 .NET 8 或更新版本時，`FlowInstance.State` 的序列化/反序列化會直接炸掉。遷移方向通常是改 JSON / protobuf / MessagePack，並規劃**現有歷史資料的轉換策略**（舊實例 blob 用舊格式讀一次、轉成新格式寫回），否則所有進行中的流程會讀不回來。

---
