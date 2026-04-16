# GROUPCARDS

## 用途

**群組卡片權限表**（Group Card Permissions）。

GROUPCARDS 定義群組/角色可見的首頁卡片，與 SYS_CARDS（卡片定義）搭配使用。使用者最終可見的卡片 = USERCARDS（個人授權）∪ GROUPCARDS（所屬群組授權）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **首頁卡片載入** | `runtimeCard_onBeforeExecuteSQL` 查詢 SYS_CARDS 時，以 GROUPCARDS 和 USERCARDS 過濾使用者可見的卡片 |
| **卡片權限管理（設計端）** | Board.cshtml 的「群組卡片」分頁，以 `groupCard` infocommand 載入群組清單，勾選後授權 |

### 關聯表

```
GROUPCARDS ──(GROUPID)──> GROUPS     （群組/角色）
           ──(CARDID)───> SYS_CARDS  （卡片定義）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `ucGroupCard_onBeforeApply()`：刪除授權（DELETE） |
| `SystemTable.Core/UserModule.cs` | `ucGroupCard_onBeforeInsert()`：新增授權（INSERT） |
| `SystemTable.Core/UserModule.cs` | `groupCard_onBeforeExecuteSQL()`：載入群組清單（ORDER BY GROUPS.GROUPID） |
| `SystemTable.Core/UserModule.cs` | `runtimeCard_onBeforeExecuteSQL()`：首頁卡片過濾（CARDID IN SELECT FROM GROUPCARDS） |
| `EEPWebClient.Core/design/server/SystemTable.json` | `groupCard` infocommand：`SELECT GROUPS.GROUPID, GROUPNAME, CARDID FROM GROUPS LEFT JOIN GROUPCARDS` |
| `EEPWebClient.Core/Views/Design/Board.cshtml` | 前端「群組卡片」管理介面（gridGroupCard） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **GROUPID** | `varchar(50)` PK | NOT NULL | 群組/角色識別碼（對應 GROUPS.GROUPID） |
| **CARDID** | `varchar(30)` PK | NOT NULL | 卡片識別碼（對應 SYS_CARDS.CARDID） |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 / Informix |
|------|-----------|-------|--------|----------------|
| GROUPID | `varchar(50)` | `varchar(50)` | `varchar2(50)` | `varchar(50)` |
| CARDID | `varchar(30)` | `varchar(30)` | `varchar2(30)` | `varchar(30)` |

---

## 主鍵

```
PRIMARY KEY (GROUPID, CARDID)
```

---

## 資料生命週期

```
授權：Board.cshtml 勾選群組
      → ucGroupCard_onBeforeInsert()
      → INSERT INTO GROUPCARDS (GROUPID, CARDID) VALUES (@GROUPID, @CARDID)

取消授權：Board.cshtml 取消勾選
      → ucGroupCard_onBeforeApply() 處理 deleted
      → DELETE FROM GROUPCARDS WHERE CARDID = @CARDID AND GROUPID = @GROUPID

首頁載入：使用者登入後載入首頁卡片
      → runtimeCard_onBeforeExecuteSQL()
      → SELECT * FROM SYS_CARDS WHERE ITEMTYPE = @Solution
        AND (CARDID IN (SELECT CARDID FROM USERCARDS WHERE USERID = @User)
         OR  CARDID IN (SELECT CARDID FROM GROUPCARDS WHERE GROUPID IN (@Groups)))
        ORDER BY SEQ_NO
```

---

## 查詢邏輯

### 群組卡片清單 — groupCard infocommand

```sql
SELECT GROUPS.GROUPID, GROUPNAME, CARDID
FROM GROUPS LEFT JOIN GROUPCARDS ON GROUPS.GROUPID = GROUPCARDS.GROUPID
  AND CARDID = @selectedCardID
ORDER BY GROUPS.GROUPID
```

以 LEFT JOIN 顯示所有群組及其授權狀態。

---

## 備註

- 與 USERCARDS 結構和操作模式完全相同，差別僅在於以 GROUPID 取代 USERID。
- Insert 和 Delete 均由 `onBeforeInsert` / `onBeforeApply` 手動組 SQL（return false 跳過預設 INSERT）。
- Informix 版本的 SQL 需加引號包裹表名與欄位名。
- 首頁卡片過濾中，群組 "00" 為預設群組，所有使用者皆屬於此群組，因此授權給 "00" 等同於全員可見。
