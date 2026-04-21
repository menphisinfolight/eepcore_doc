# FLOWCOMMENTDETAIL

## 用途

**流程評論接收者明細表**（Flow Comment Detail）。

FLOWCOMMENTDETAIL 儲存每則流程評論的接收者記錄，與 FLOWCOMMENT 為一對多關係。每當使用者發送評論並指定接收者（使用者或角色）時，系統為每個接收對象建立一筆明細，並透過 IsRead 欄位追蹤已讀狀態。

### 使用場景

| 場景 | 說明 |
|------|------|
| **評論通知** | CommentActivity.Invoke() 為每個接收者（Users/Roles）建立 CommentDetailActivity，寫入明細 |
| **待辦評論查詢** | SelectComment(pageSize, pageIndex) 以 Detail LEFT JOIN Comment 查出使用者/角色收到的評論 |
| **流程內評論檢視** | SelectComment(instanceID) 查出該實例所有評論，並從 Detail 組合 SendToName 顯示接收者姓名 |
| **標記已讀** | Read(replyIDs) 批次更新 IsRead = true，用於前端標記已讀 |
| **回覆觸發已讀** | Insert(CommentActivity) 時若帶有 ReplyID，會先呼叫 Read() 將被回覆的明細標為已讀 |

### 關聯表

```
FLOWCOMMENTDETAIL ──(InstanceID + ActivityID)──> FLOWCOMMENT  （評論主表）
                  ──(InstanceID)──────────────> FLOWINSTANCE  （流程實例）
                  ──(RoleID)──────────────────> GROUPS        （角色/群組）
                  ──(UserID)──────────────────> USERS         （使用者）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowCommentHelper.cs` | 資料存取：Insert(CommentDetailActivity) / SelectComment / Read |
| `EEP.Flow/Activities/CommentActivity.cs` | CommentActivity.Invoke() 建立 CommentDetailActivity 並寫入明細 |
| `EEP.Flow/Activities/CommentActivity.cs` | CommentDetailActivity.Invoke() 呼叫 FlowCommentHelper.Insert(this) |
| `EEP.Flow/Instance.cs` | Instance.Comment() 建立 CommentActivity 並啟動流程 |
| `EEP.Flow/API.cs` | API.Comment() / ReadComment() / QueryComment() |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **InstanceID** | `nvarchar(50)` | NOT NULL | 流程實例編號 |
| **ActivityID** | `nvarchar(50)` | NOT NULL | 評論活動編號（對應 FLOWCOMMENT.ActivityID） |
| **ReplyID** | `nvarchar(50)` | NOT NULL | 接收者明細的唯一識別碼（CommentDetailActivity 的 GUID） |
| **RoleID** | `nvarchar(50)` | NULL | 接收角色編號（對應 GROUPS.GROUPID），與 UserID 擇一填入 |
| **RoleName** | `nvarchar(50)` | NULL | 接收角色名稱（寫入時由 GetGroupName() 解析） |
| **UserID** | `nvarchar(50)` | NULL | 接收使用者帳號（對應 USERS.USERID），與 RoleID 擇一填入 |
| **UserName** | `nvarchar(50)` | NULL | 接收使用者姓名（寫入時由 GetUserName() 解析） |
| **IsRead** | `bit` | NULL | 是否已讀（Insert 時預設 false，ReadComment 時更新為 true） |
| **ProjectID** | `nvarchar(50)` | NULL | 所屬解決方案編號（後期擴充欄位） |

### 跨資料庫差異

五個 DB CREATE TABLE 都完整。`IsRead` 型別跨 DB 不統一（bit / CHAR(1) / DECIMAL(1,0)）—— 讀取端需處理多種表示。

👉 升級 ALTER 支援矩陣、手動補欄位 SQL：**問題_SP7_跨資料庫欄位差異盤點**

---

## 索引

```
INDEX ROLEINDEX ON FlowCommentDetail (RoleID)
INDEX USERINDEX ON FlowCommentDetail (UserID)
```

無主鍵定義。同一 ActivityID 可有多筆明細（一則評論發給多個接收者）。

---

## 資料生命週期

```
建立：CommentActivity.Invoke()
      → 拆分 Users/Roles 為個別 CommentDetailActivity
      → FlowCommentHelper.Insert(CommentDetailActivity)
      → 寫入 InstanceID, ActivityID(父評論), ReplyID(自身GUID), Role/User, IsRead=false

已讀：API.ReadComment(replyIDs)
      → FlowCommentHelper.Read(replyIDs)
      → UPDATE FlowCommentDetail SET IsRead = true WHERE ReplyID IN (...)

回覆觸發已讀：FlowCommentHelper.Insert(CommentActivity)
      → 若 ReplyID 不為空 → 先呼叫 Read([replyID]) 將被回覆項標為已讀
```

---

## 查詢邏輯

### 待辦評論查詢（分頁）— SelectComment(pageSize, pageIndex)

```sql
SELECT ReplyID, FlowComment.*
FROM FlowCommentDetail
LEFT JOIN FlowComment
  ON FlowCommentDetail.InstanceID = FlowComment.InstanceID
 AND FlowCommentDetail.ActivityID = FlowComment.ActivityID
WHERE FlowCommentDetail.InstanceID IN (SELECT InstanceID FROM FlowToDo)
  AND (RoleID IN (@r0, @r1, ...) OR UserID = @userid)
ORDER BY Datetime DESC
```

- 僅顯示仍有待辦的流程評論（InstanceID IN FlowToDo）
- 以使用者所屬群組（Groups）和個人帳號（UserID）過濾

### 流程內評論檢視 — SelectComment(instanceID)

查詢 FLOWCOMMENT 主表後，再查 FLOWCOMMENTDETAIL 明細，將每則評論的接收者組合為 `SendToName`（角色名優先，無角色則取使用者名），並判斷 `CanReply`（發送者不等於當前使用者時可回覆）。

---

## 備註

- RoleID 和 UserID 為擇一填入：若接收對象是角色則填 RoleID/RoleName，若是個人則填 UserID/UserName。
- ReplyID 是 CommentDetailActivity 的 GUID，用於唯一識別每筆明細，也是 ReadComment 批次已讀的操作鍵。
- ProjectID 為後期擴充欄位，SQL 腳本中有 ALTER TABLE 補欄位邏輯。
- DB2/Informix 環境下表名會經過 `MarkTable()` 處理（加引號）。
- Update（修改評論）和 Delete（刪除評論）方法在程式碼中已被註解掉，目前不支援修改或刪除評論。
