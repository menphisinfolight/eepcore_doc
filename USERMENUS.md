# USERMENUS

## 用途

**使用者選單權限表**（User Menu Permissions）。

USERMENUS 定義個別使用者可以存取哪些選單（MENUTABLE），以及該使用者在該選單上的新增/修改/刪除權限。與 GROUPMENUS 為對偶表，結構完全相同，僅主鍵從 GROUPID 改為 USERID。

### 使用場景

| 場景 | 說明 |
|------|------|
| **選單載入過濾** | `runtimeMenu_onBeforeExecuteSQL()` 以 `MENUID IN (SELECT MENUID FROM USERMENUS WHERE USERID=@User)` 過濾使用者可見選單 |
| **使用者權限匯出** | `SecurityProvider.ExportUserAccess()` 以 UNION 合併 USERMENUS 和 GROUPMENUS 的 ALLOWADD/ALLOWUPDATE/ALLOWDELETE |
| **選單→使用者管理** | `menuUser` infocommand：以選單為主查看已授權的使用者 |
| **使用者→選單管理** | `menuUsers` infocommand：以使用者為主查看已授權的選單 |
| **未授權使用者查詢** | `userMenu` infocommand：LEFT JOIN + WHERE MENUID IS NULL 篩出未授權的使用者 |

### 權限合併規則

使用者最終選單權限 = USERMENUS（直接授權）∪ GROUPMENUS（透過 USERGROUPS 群組繼承）。`runtimeMenu_onBeforeExecuteSQL` 以 OR 條件合併兩者。

### 關聯表

```
USERMENUS ──(USERID)──> USERS      （使用者帳號）
          ──(MENUID)──> MENUTABLE  （選單定義）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `runtimeMenu_onBeforeExecuteSQL()`：選單載入過濾（USERMENUS OR GROUPMENUS） |
| `SystemTable.Core/UserModule.cs` | `ucUserMenu_onBeforeApply()`：DELETE 授權；`ucUserMenu_onBeforeInsert()`：INSERT 授權（先刪後寫） |
| `SystemTable.Core/UserModule.cs` | `userMenu_onBeforeExecuteSQL()`：LEFT JOIN + WHERE MENUID IS NULL 查未授權使用者 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUserAccess()`：UNION 合併 USERMENUS + GROUPMENUS 匯出完整權限 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `menuUser`：選單→使用者視角；`menuUsers`：使用者→選單視角；`userMenu`：未授權使用者查詢 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **MENUID** | `nvarchar(30)` PK | NOT NULL | 選單識別碼（對應 MENUTABLE.MENUID） |
| **ALLOWADD** | `char(1)` | NULL | 允許新增（`Y`/`N`/空），後期擴充欄位 |
| **ALLOWUPDATE** | `char(1)` | NULL | 允許修改（`Y`/`N`/空），後期擴充欄位 |
| **ALLOWDELETE** | `char(1)` | NULL | 允許刪除（`Y`/`N`/空），後期擴充欄位 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar/nvarchar`。無特殊型別差異。

---

## 主鍵

```
PRIMARY KEY (USERID, MENUID)
```

---

## 資料生命週期

```
授權（選單→使用者視角）：Board.cshtml → ucUserMenu
      → ucUserMenu_onBeforeInsert() → DELETE 同鍵 + INSERT
      → INSERT INTO USERMENUS (USERID, MENUID)

授權（使用者→選單視角）：Board.cshtml → ucMenuUsers
      → ucMenuUsers_onBeforeInsert() → return true（使用預設 INSERT）

取消授權：Board.cshtml
      → ucUserMenu_onBeforeApply() / ucMenuUsers_onBeforeApply()
      → DELETE FROM USERMENUS WHERE MENUID = @MENUID AND USERID = @USERID

選單載入：使用者登入後
      → runtimeMenu_onBeforeExecuteSQL()
      → SELECT * FROM MENUTABLE WHERE ITEMTYPE = @Solution
        AND (MENUID IN (SELECT MENUID FROM USERMENUS WHERE USERID = @User)
         OR  MENUID IN (SELECT MENUID FROM GROUPMENUS WHERE GROUPID IN (@Groups)))
        ORDER BY SEQ_NO, MENUID
```

---

## 備註

- ALLOWADD/ALLOWUPDATE/ALLOWDELETE 為後期擴充欄位，SQL 腳本中有 ALTER TABLE 補欄位邏輯。
- `ucMenuUsers_onBeforeApply` 的非 Informix 版本中 DELETE 邏輯被註解掉了（使用預設行為），Informix 版本則手動組 SQL。
- 更細緻的控制項權限由 USERMENUCONTROL 控制。
- EveryOne 群組（GROUPID='00'）在 runtimeMenu 查詢時自動包含，不需在 USERMENUS 中設定。
