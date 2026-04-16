# GROUPMENUS

## 用途

**群組選單權限表**（Group Menu Permissions）。

GROUPMENUS 定義每個群組/角色可以存取哪些選單（MENUTABLE），以及該群組在該選單上的新增/修改/刪除權限。是 EEP Core 權限模型「使用者 → 群組 → 選單權限」中的關鍵連結表。與 USERMENUS 為對偶表，結構完全相同，僅主鍵從 USERID 改為 GROUPID。

### 使用場景

| 場景 | 說明 |
|------|------|
| **選單載入過濾** | `runtimeMenu_onBeforeExecuteSQL()` 以 `MENUID IN (SELECT MENUID FROM GROUPMENUS WHERE GROUPID IN (@Groups))` 過濾 |
| **使用者權限匯出** | `SecurityProvider.ExportUserAccess()` 以 UNION 合併 USERMENUS 和 GROUPMENUS |
| **選單→群組管理** | `menuGroup` infocommand：以選單為主查看已授權的群組 |
| **群組→選單管理** | `menuGroups` infocommand：以群組為主查看已授權的選單（含 CAPTION, MODULETYPE） |
| **未授權群組查詢** | `groupMenu` infocommand：LEFT JOIN + WHERE MENUID IS NULL 篩出未授權的群組 |
| **新選單自動授權** | `ucMenu_onBeforeInsert()` 新增選單時自動為 EveryOne（'00'）群組新增授權 |
| **方案重新命名** | `SecurityProvider.RenameSolution()` 不直接影響 GROUPMENUS（GROUPMENUS 無 ITEMTYPE 欄位） |

### 權限合併規則

使用者最終選單權限 = USERMENUS（直接授權）∪ GROUPMENUS（透過 USERGROUPS 群組繼承），`runtimeMenu_onBeforeExecuteSQL` 以 OR 條件合併。

### 關聯表

```
GROUPMENUS ──(GROUPID)──> GROUPS     （群組/角色）
           ──(MENUID)───> MENUTABLE  （選單定義）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `runtimeMenu_onBeforeExecuteSQL()`：選單載入過濾（GROUPMENUS OR USERMENUS） |
| `SystemTable.Core/UserModule.cs` | `ucMenu_onBeforeInsert()`：新選單自動授權給 EveryOne |
| `SystemTable.Core/UserModule.cs` | `ucGroupMenu_onBeforeApply/Insert()`、`ucMenuGroup` 等：群組選單授權管理 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUserAccess()`：UNION 合併權限匯出 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `menuGroup`：選單→群組；`menuGroups`：群組→選單；`groupMenu`：未授權群組查詢 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **GROUPID** | `varchar(50)` PK | NOT NULL | 群組/角色識別碼（對應 GROUPS.GROUPID） |
| **MENUID** | `nvarchar(30)` PK | NOT NULL | 選單識別碼（對應 MENUTABLE.MENUID） |
| **ALLOWADD** | `char(1)` | NULL | 允許新增（`Y`/`N`/空），後期擴充欄位 |
| **ALLOWUPDATE** | `char(1)` | NULL | 允許修改（`Y`/`N`/空），後期擴充欄位 |
| **ALLOWDELETE** | `char(1)` | NULL | 允許刪除（`Y`/`N`/空），後期擴充欄位 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。GROUPID 早期為 `varchar(20)`，後透過 ALTER TABLE 擴展為 `varchar(50)`。

---

## 主鍵

```
PRIMARY KEY (GROUPID, MENUID)
```

---

## 預設資料

```sql
INSERT INTO GROUPMENUS(GROUPID, MENUID) VALUES('00', '0');
```

EveryOne 群組預設擁有根選單（MENUID='0'）的存取權。

---

## 資料生命週期

```
自動授權（新選單）：ucMenu_onBeforeInsert()
      → DELETE FROM GROUPMENUS WHERE MENUID = @newMenuID
      → INSERT INTO GROUPMENUS (GROUPID, MENUID) VALUES ('00', @newMenuID)

授權（選單→群組視角）：Board.cshtml → ucGroupMenu
      → ucGroupMenu_onBeforeInsert()：先 DELETE 同鍵 → INSERT
      → ucGroupMenu_onBeforeApply()：DELETE

授權（群組→選單視角）：Board.cshtml → ucMenuGroup
      → ucMenuGroups_onBeforeInsert()：return true（預設 INSERT）

選單載入：runtimeMenu_onBeforeExecuteSQL()
      → SELECT * FROM MENUTABLE WHERE ITEMTYPE = @Solution
        AND (MENUID IN (SELECT MENUID FROM USERMENUS WHERE USERID = @User)
         OR  MENUID IN (SELECT MENUID FROM GROUPMENUS WHERE GROUPID IN (@Groups)))
        ORDER BY SEQ_NO, MENUID
```

---

## 備註

- ALLOWADD/ALLOWUPDATE/ALLOWDELETE 為後期擴充欄位（SQL 腳本中有 ALTER TABLE 補欄位邏輯）。
- 新增選單時自動為 EveryOne 群組授權，確保新選單對所有人可見（除非後續移除）。
- 更細緻的控制項權限由 GROUPMENUCONTROL 控制（但該表目前在程式碼中無引用）。
- `ucMenuGroups_onBeforeInsert` 非 Informix 版本中的 DELETE 邏輯被註解掉，使用預設行為；Informix 版本手動組 SQL。
