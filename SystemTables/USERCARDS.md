# USERCARDS

## 用途

**使用者卡片權限表**（User Card Permissions）。

USERCARDS 定義個別使用者可見的首頁卡片，與 SYS_CARDS（卡片定義）搭配使用。使用者最終可見的卡片 = USERCARDS（個人授權）∪ GROUPCARDS（所屬群組授權）。

### 使用場景

| 場景 | 說明 |
|------|------|
| **首頁卡片載入** | `runtimeCard_onBeforeExecuteSQL` 查詢 SYS_CARDS 時，以 USERCARDS 和 GROUPCARDS 過濾使用者可見的卡片 |
| **卡片權限管理（設計端）** | Board.cshtml 的「使用者卡片」分頁，以 `userCard` infocommand 載入未授權的使用者清單，勾選後授權 |
| **卡片權限查看** | `cardUser` infocommand 以 LEFT JOIN 查詢已授權使用者與卡片的對應關係 |

### 關聯表

```
USERCARDS ──(USERID)──> USERS      （使用者帳號）
          ──(CARDID)──> SYS_CARDS  （卡片定義）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `ucUserCard_onBeforeApply()`：刪除授權（DELETE） |
| `SystemTable.Core/UserModule.cs` | `ucUserCard_onBeforeInsert()`：新增授權（INSERT） |
| `SystemTable.Core/UserModule.cs` | `userCard_onBeforeExecuteSQL()`：載入未授權使用者（WHERE CARDID IS NULL） |
| `SystemTable.Core/UserModule.cs` | `runtimeCard_onBeforeExecuteSQL()`：首頁卡片過濾（CARDID IN SELECT FROM USERCARDS） |
| `EEPWebClient.Core/design/server/SystemTable.json` | `cardUser` infocommand：`SELECT USERS.USERID, USERNAME, CARDID FROM USERCARDS LEFT JOIN USERS` |
| `EEPWebClient.Core/design/server/SystemTable.json` | `userCard` infocommand：`SELECT USERS.USERID, USERNAME, CARDID FROM USERS LEFT JOIN USERCARDS` |
| `EEPWebClient.Core/Views/Design/Board.cshtml` | 前端「使用者卡片」管理介面（gridUserCard） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **CARDID** | `varchar(30)` PK | NOT NULL | 卡片識別碼（對應 SYS_CARDS.CARDID） |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 / Informix |
|------|-----------|-------|--------|----------------|
| USERID | `varchar(20)` | `varchar(20)` | `varchar2(20)` | `varchar(20)` |
| CARDID | `varchar(30)` | `varchar(30)` | `varchar2(30)` | `varchar(30)` |

---

## 主鍵

```
PRIMARY KEY (USERID, CARDID)
```

---

## 資料生命週期

```
授權：Board.cshtml 勾選使用者
      → ucUserCard_onBeforeInsert()
      → INSERT INTO USERCARDS (USERID, CARDID) VALUES (@USERID, @CARDID)

取消授權：Board.cshtml 取消勾選
      → ucUserCard_onBeforeApply() 處理 deleted
      → DELETE FROM USERCARDS WHERE CARDID = @CARDID AND USERID = @USERID

首頁載入：使用者登入後載入首頁卡片
      → runtimeCard_onBeforeExecuteSQL()
      → SELECT * FROM SYS_CARDS WHERE ITEMTYPE = @Solution
        AND (CARDID IN (SELECT CARDID FROM USERCARDS WHERE USERID = @User)
         OR  CARDID IN (SELECT CARDID FROM GROUPCARDS WHERE GROUPID IN (@Groups)))
        ORDER BY SEQ_NO
```

---

## 查詢邏輯

### 未授權使用者查詢 — userCard infocommand

```sql
SELECT USERS.USERID, USERNAME, CARDID
FROM USERS LEFT JOIN USERCARDS ON USERS.USERID = USERCARDS.USERID
  AND CARDID = @selectedCardID
WHERE CARDID IS NULL
ORDER BY USERS.USERID
```

以 LEFT JOIN + WHERE CARDID IS NULL 篩出「尚未被授權該卡片」的使用者。

### 已授權使用者查詢 — cardUser infocommand

```sql
SELECT USERS.USERID, USERNAME, CARDID
FROM USERCARDS LEFT JOIN USERS ON USERS.USERID = USERCARDS.USERID
```

透過 infodatasource 以 SYS_CARDS.CARDID 關聯，顯示每張卡片已授權的使用者。

---

## 備註

- Insert 和 Delete 均由 `onBeforeInsert` / `onBeforeApply` 手動組 SQL，而非使用 UpdateComponent 預設行為（return false 表示跳過預設 INSERT）。
- Informix 版本的 SQL 需加引號包裹表名與欄位名。
- 首頁卡片過濾中，群組 "00" 為預設群組，所有使用者皆屬於此群組。
