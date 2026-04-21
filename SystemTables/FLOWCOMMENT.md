# FLOWCOMMENT

## 用途

**流程評論表**（Flow Comment）。

FLOWCOMMENT 儲存流程步驟中的評論/意見交流記錄，支援附件上傳。簽核者可以在流程中留言討論，其他人可回覆。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **流程內部討論** | 簽核者在簽核過程中針對表單內容進行意見交流 |
| **@提及通知** | 評論可以指定發送給特定使用者或角色（類似 @mention） |
| **附件討論** | 支援上傳附件進行補充說明 |
| **回覆追蹤** | 支援回覆特定評論，形成討論串 |
| **已讀標記** | 接收者可標記評論為已讀 |

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **FlowID** | `nvarchar(50)` | 流程定義 ID |
| **InstanceID** | `nvarchar(50)` | 流程實例 ID |
| **FlowText** | `nvarchar(50)` | 流程名稱 |
| **ActivityID** | `nvarchar(50)` | 活動/步驟 ID |
| **SenderID** | `nvarchar(50)` | 發送者帳號 |
| **SenderName** | `nvarchar(50)` | 發送者姓名 |
| **Datetime** | `datetime` | 發送時間 |
| **Remark** | `nvarchar(2048)` | 評論內容 |
| **Attachments** | `nvarchar(2048)` | 附件路徑 |
| **Parameter** | `nvarchar(max)` | 參數 |
| **PActivityID** | `nvarchar(50)` | 前一步驟活動 ID |
| **ProjectID** | `nvarchar(50)` | 專案 ID |

### 跨資料庫差異

五個 DB 都含完整 12 欄。主要差異：

| 欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|------|-------|--------|-------|-----|----------|
| `Datetime` | `datetime` | `date` | `datetime` | `TIMESTAMP` | `DATETIME YEAR TO SECOND` |
| `Parameter` | `nvarchar(max)` | `clob` | `text` | `nvarchar(9800)` | `LVARCHAR(...)` |
| `Remark` / `Attachments` | `nvarchar(2048)` | `varchar2(2048)` | `nvarchar(2048)` | `nvarchar(2048)` | `LVARCHAR(2048)` |

#### SP7 升級 ALTER ADD 矩陣

| 新增欄位 | MSSQL | Oracle | MySQL | DB2 | Informix |
|---------|:-:|:-:|:-:|:-:|:-:|
| `ProjectID` | ✅ | ❌ | ❌ | ❌ | ❌ |

其他 DB 的 CREATE TABLE 已含 `ProjectID`，舊版升級時需手動補。

---

## 關聯表：FLOWCOMMENTDETAIL

FLOWCOMMENTDETAIL 儲存評論的接收者明細，記錄哪些使用者或角色收到評論通知。

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **InstanceID** | `nvarchar(50)` | 流程實例 ID |
| **ActivityID** | `nvarchar(50)` | 評論活動 ID |
| **ReplyID** | `nvarchar(50)` | 回覆活動 ID（子評論識別） |
| **RoleID** | `nvarchar(50)` | 接收角色 ID |
| **RoleName** | `nvarchar(50)` | 接收角色名稱 |
| **UserID** | `nvarchar(50)` | 接收使用者 ID |
| **UserName** | `nvarchar(50)` | 接收使用者名稱 |
| **IsRead** | `bit` | 是否已讀 |

---

## 索引

```
INDEX MAININDEX ON FlowComment (InstanceID, ActivityID)
```

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowCommentHelper.cs` | 評論資料庫操作類別 |
| `EEP.Flow/Activities/CommentActivity.cs` | 評論活動定義（CommentActivity、CommentDetailActivity） |
| `EEP.Flow/API.cs` | 流程 API 層（Comment、ReadComment、QueryComment） |

---

## API 方法說明

| 方法 | 說明 |
|------|------|
| `Comment(instanceID, activityID, users, roles, remark, attachments, replyID)` | 發送評論 |
| `ReadComment(replyIDs)` | 標記評論為已讀 |
| `QueryComment(pageSize, pageIndex, queryParam)` | 查詢評論列表（分頁） |
| `QueryComment(instanceID)` | 查詢指定流程的所有評論 |

---

## 評論活動類型

### CommentActivity（主評論）
- 繼承 `ContinuedActivity`
- 屬性：`Roles`（角色清單）、`Users`（使用者清單）、`Attachments`（附件）、`ReplyID`（回覆對象）
- 執行時會建立對應的 `CommentDetailActivity` 給每個接收者

### CommentDetailActivity（接收明細）
- 繼承 `ContinuedActivity`
- 記錄單個接收者的評論通知
- 屬性：`Role`、`User`、`ParentActivityID`

---

## 備註

- FLOWCOMMENT 與 FLOWCOMMENTDETAIL 為一對多關係，主表記錄評論內容，明細表記錄接收者。
- 評論支援多接收者（同時發給多個使用者或角色）。
- `ReplyID` 用於實現回覆功能，指向父評論的 ActivityID。
- `IsRead` 標記在 FLOWCOMMENTDETAIL 中，每個接收者獨立計算已讀狀態。
- 附件路徑以特定格式儲存（如多個附件以分隔符號串接）。
