# FLOWHISTORY

## 用途

**流程簽核歷史表**（Flow History）。

FLOWHISTORY 記錄流程每一步的簽核歷史，包含簽核者、角色、處理狀態、耗時等。是流程追蹤、審計與效能分析的核心表。每次活動執行（送出、退回、加簽、轉簽等）都會新增一筆記錄。

### 使用場景

| 場景 | 說明 |
|------|------|
| **簽核記錄** | 活動執行時，FlowHistoryHelper.Insert() 寫入當次簽核的完整資訊 |
| **流程預覽** | Instance.Preview() 查詢該實例的所有簽核歷史，顯示於前端 |
| **結案查詢** | API.QueryEnd() 查詢使用者參與過且已結案的流程 |
| **流程分析** | FlowAnalysisProvider 統計流程耗時、瓶頸環節、效率分析 |
| **資訊權限** | InfoCommand 以 FlowHistory 的 RoleID 控制資料可見範圍（SecStyle.FlowHistory） |

### 關聯表

```
FLOWHISTORY ──(InstanceID)──> FLOWINSTANCE  （流程實例）
            ──(InstanceID)──> FLOWTODO      （待辦事項）
            ──(InstanceID)──> FLOWNOTIFY    （通知）
            ──(InstanceID)──> FLOWCOMMENT   （簽核意見）
            ──(RoleID)─────> GROUPS         （角色/群組）
            ──(UserID)─────> USERS          （使用者）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowHistoryHelper.cs` | 資料存取：Insert（5 種重載）/ Delete / SelectDetail / SelectEnd |
| `EEP.Flow/Instance.cs` | Delete() 刪除歷史、Preview() 查詢歷史 |
| `EEP.Flow/API.cs` | QueryEnd() / QueryAllEnd() / QueryHistory() / QueryToDo() |
| `EEPGlobal.Core/Provider/FlowAnalysisProvider.cs` | 流程分析：統計、耗時、瓶頸 |
| `EEPServerTools.Core/Components/InfoCommand.cs` | SecStyle.FlowHistory 資訊權限控制 |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.flow.js` | 前端流程操作介面 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FlowID** | `nvarchar(50)` | NOT NULL | 流程定義編號 |
| **InstanceID** | `nvarchar(50)` | NOT NULL | 流程實例編號 |
| **FlowText** | `nvarchar(50)` | NOT NULL | 流程名稱 |
| **ActivityID** | `nvarchar(50)` | NOT NULL | 活動/步驟編號 |
| **ActivityText** | `nvarchar(50)` | NOT NULL | 活動/步驟名稱 |
| **RoleID** | `nvarchar(50)` | NULL | 簽核角色編號（對應 GROUPS.GROUPID） |
| **RoleName** | `nvarchar(50)` | NULL | 簽核角色名稱 |
| **UserID** | `nvarchar(50)` | NULL | 簽核者帳號（對應 USERS.USERID） |
| **UserName** | `nvarchar(50)` | NULL | 簽核者姓名，代理簽核時後綴 `(*)` |
| **Status** | `int` | NOT NULL | 處理狀態（見下方列舉） |
| **Datetime** | `datetime` | NOT NULL | 簽核操作時間 |
| **Remark** | `nvarchar(2048)` | NULL | 簽核意見/備註 |
| **Parameter** | `nvarchar(max)` | NULL | 流程參數（JSON 格式） |
| **StartDatetime** | `datetime` | NULL | 該活動的開始時間 |
| **ProjectID** | `nvarchar(50)` | NULL | 所屬解決方案編號（後期擴充） |
| **Duration** | `decimal(8,2)` | NULL | 處理耗時（天數），由 Datetime - StartDatetime 計算（後期擴充） |
| **ExpDatetime** | `datetime` | NULL | 預期完成時間（後期擴充） |

### Status 狀態列舉

| 值 | 狀態 | 說明 |
|----|------|------|
| 0 | 啟動 | 流程啟動 |
| 1 | 審核（Submit） | 正常送出簽核 |
| 2 | 退回（Return） | 退回上一步 |
| 3 | 取回（Retake） | 送出後取回 |
| 5 | 加簽（Plus） | 加簽給其他人 |
| 6 | 轉簽（Transfer） | 轉簽給其他人 |
| 7 | 作廢（Reject） | 流程作廢 |
| 8 | 預啟動（Prepare） | 預先啟動 |
| 9 | 結案（End） | 流程完成 |
| 11 | 通知（Notify） | 通知活動 |

---

## 索引

```
INDEX ROLEINDEX ON FlowHistory (RoleID, Datetime)
INDEX USERINDEX ON FlowHistory (UserID, Datetime)
```

用於加速按角色或使用者查詢簽核歷史。

---

## Insert 重載

FlowHistoryHelper 提供 5 種 Insert 重載，對應不同活動類型：

| 重載 | 活動類型 | 說明 |
|------|----------|------|
| Insert(StandActivity) | 標準簽核活動 | 含完整簽核資訊，代理簽核時 UserName 加 `(*)` |
| Insert(DetailResultActivity) | 詳細結果活動 | 流程結果活動的歷史 |
| Insert(SystemActivity) | 系統活動 | 系統自動執行的活動，UserID/UserName 為空 |
| Insert(RootActivity) | 根活動 | 流程啟動時的根活動記錄 |
| Insert(NotifyActivity) | 通知活動 | 通知類型活動，Status = 11 |

---

## 備註

- 此表**無主鍵**，同一 InstanceID 會有多筆記錄（每個簽核步驟一筆）。
- ProjectID、Duration、ExpDatetime 為後期擴充欄位，建表腳本中有 ALTER TABLE 補欄位的邏輯。
- Duration 的計算方式為 `(Service.Now - activity.StartDateTime).TotalDays`，僅在有 ExpDatetime 時才計算。
- 流程分析（FlowAnalysisProvider）透過 FLOWHISTORY 自我 JOIN，以 Status=1（審核）和 Status=9（結案）配對計算流程總耗時。
- SecStyle.FlowHistory 模式下，InfoCommand 以使用者所屬群組的 RoleID 過濾可見的 InstanceID。
