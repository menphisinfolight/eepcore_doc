# GROUPS

## 用途

**群組/角色主檔**（Groups / Roles Master）。

GROUPS 儲存 EEP Core 系統中的群組與角色定義。EEP 的權限模型採「使用者 → 群組/角色 → 選單權限」三層架構，GROUPS 是中間層的核心表。群組（ISROLE ≠ Y）代表部門/組織單位，角色（ISROLE = Y）代表功能角色。

### 使用場景

| 場景 | 說明 |
|------|------|
| **選單權限繼承** | 透過 GROUPMENUS 設定群組可用的選單，使用者透過 USERGROUPS 繼承 |
| **卡片權限** | 透過 GROUPCARDS 設定群組可見的首頁卡片 |
| **登入載入** | `AccountProvider.SetClientInfo()` 查詢 `USERGROUPS LEFT JOIN GROUPS`，區分 Groups 和 Roles（ISROLE='Y'） |
| **安全查詢** | `SecurityProvider.ExportUser()` 組合 GROUPNAMES（群組名）和 ROLENAMES（角色名） |
| **組織架構** | SYS_ORGROLES 關聯 GROUPID 到組織角色，SYS_ORG 的 ORG_MAN 透過 USERGROUPS 關聯 |
| **MSAD 整合** | MSAD 欄位設定 Active Directory 群組對應名稱 |
| **匯入/匯出** | `ucGroup_onBeforeApply()` 支援 replace（全刪重建）和 insert（updateIfExists）兩種匯入模式 |
| **匯入時重建成員** | `ucGroup_onAfterInsert()` / `ucGroup_onBeforeUpdate()` 匯入群組時一併 DELETE + INSERT 重建 USERGROUPS 成員 |

### 關聯表

```
USERS ──(USERGROUPS)──> GROUPS ──(GROUPMENUS)──> MENUTABLE
                                ──(GROUPCARDS)──> SYS_CARDS
                                ──(SYS_ORGROLES)──> SYS_ORG
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `SetClientInfo()`：登入後查詢使用者所屬群組/角色 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUser()`/`ExportUserAccess()`/`ExportGroupUser()`：查詢群組權限 |
| `SystemTable.Core/UserModule.cs` | `ucGroup_onBeforeApply()`：匯入模式處理；`ucGroup_onAfterInsert()`：重建成員 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `group` infocommand：`SELECT * FROM GROUPS`，keys: `GROUPID` |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **GROUPID** | `varchar(50)` PK | NOT NULL | 群組/角色識別碼（如 `00`=EveryOne） |
| **GROUPNAME** | `nvarchar(50)` | NULL | 群組/角色顯示名稱 |
| **DESCRIPTION** | `nvarchar(100)` | NULL | 群組/角色描述 |
| **MSAD** | `nvarchar(50)` | NULL | Active Directory 群組對應名稱 |
| **ISROLE** | `char(1)` | NULL | 是否為角色（`Y`=角色，`N`/空=群組/部門） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar/nvarchar`。GROUPID 欄位早期為 `varchar(20)`，後透過 ALTER TABLE 擴展為 `varchar(50)`。

---

## 主鍵

```
PRIMARY KEY (GROUPID)
```

---

## 預設資料

```sql
INSERT INTO GROUPS(GROUPID, GROUPNAME, MSAD) VALUES('00', 'EveryOne', 'N');
```

`EveryOne` 群組代表「所有使用者」，不需要在 USERGROUPS 中明確加入，安全查詢和選單載入時會自動包含（`groups.Add("00")`）。

---

## 資料生命週期

```
建立：設計端 Board.cshtml → ucGroup → INSERT INTO GROUPS
      → ucMenu_onBeforeInsert() 自動為新選單授權給 EveryOne 群組

匯入（replace 模式）：ucGroup_onBeforeApply()
      → DELETE ALL FROM GROUPS → INSERT 所有群組
      → ucGroup_onAfterInsert() → DELETE + INSERT 重建 USERGROUPS 成員

匯入（insert 模式）：ucGroup_onBeforeApply()
      → updateIfExists = true → INSERT 或 UPDATE
      → ucGroup_onBeforeUpdate() → 重建 USERGROUPS 成員

登入載入：AccountProvider.SetClientInfo()
      → SELECT * FROM USERGROUPS LEFT JOIN GROUPS WHERE USERID = @user
      → clientInfo.Groups = 所有 GROUPID
      → clientInfo.Roles = ISROLE='Y' 的 GROUPID
```

---

## 備註

- GROUPS 同時承載「群組」（部門/組織）和「角色」（功能權限）兩種概念，透過 ISROLE 區分。
- 新增選單時（`ucMenu_onBeforeInsert`），會自動為 EveryOne 群組（'00'）新增 GROUPMENUS 授權。
- 匯入群組時會一併重建該群組的 USERGROUPS 成員關係（DELETE + INSERT）。
- MSAD 欄位早期為必填（INSERT 時設 'N'），後透過 ALTER TABLE 改為 NULL。
