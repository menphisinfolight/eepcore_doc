# SYS_ORGROLES

## 用途

**組織角色關聯表**（Organization-Role Assignment）。

SYS_ORGROLES 定義組織單位與角色的關聯，為每個組織單位指派功能角色（GROUPS 中 ISROLE='Y' 的記錄）。流程引擎透過此表找到角色所屬的組織，進而查詢上層主管。

### 使用場景

| 場景 | 說明 |
|------|------|
| **登入載入 OrgRoles** | `AccountProvider.GetOrgRoles()` 查詢使用者管轄組織下的所有 ROLE_ID，加入 ClientInfo.OrgRoles |
| **流程主管查詢** | `FlowProvider.GetManagerID()` 以 ROLE_ID 查 SYS_ORGROLES 找到 ORG_NO，再從 SYS_ORG 取得上層主管 |
| **組織樹顯示** | `org_onBeforeExecuteSQL()` Runtime 模式以 UNION 顯示 SYS_ORGROLES（TYPE='U'）的角色節點 |
| **角色→組織管理** | `roleOrg` infocommand：LEFT JOIN GROUPS 顯示角色清單及其所屬組織 |
| **組織→角色管理** | `orgRole` infocommand：LEFT JOIN GROUPS 顯示組織下的角色明細 |

### 關聯表

```
SYS_ORGROLES ──(ORG_NO)───> SYS_ORG   （組織架構）
             ──(ROLE_ID)──> GROUPS     （角色，ISROLE='Y'）
             ──(ORG_KIND)─> SYS_ORGKIND（組織類型，冗餘欄位）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `GetOrgRoles()`：查詢管轄組織下的 ROLE_ID |
| `EEPServerTools.Core/Adapter/Flow.cs` | `GetManagerID()`：以 ROLE_ID 查 SYS_ORGROLES 找到 ORG_NO |
| `SystemTable.Core/UserModule.cs` | `org_onBeforeExecuteSQL()`：UNION 組織+角色；`ucRoleOrg_onBeforeApply/Insert()`：角色組織管理 |
| `SystemTable.Core/UserModule.cs` | `roleOrg_onBeforeExecuteSQL()`：過濾 ISROLE='Y' 的群組 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `orgRole`：組織→角色明細（infodatasource 以 ORG_NO 關聯 org）；`roleOrg`：角色→組織（LEFT JOIN） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ORG_NO** | `nvarchar(8)` PK | NOT NULL | 組織編號（對應 SYS_ORG.ORG_NO） |
| **ROLE_ID** | `varchar(50)` PK | NOT NULL | 角色識別碼（對應 GROUPS.GROUPID，ISROLE='Y'） |
| **ORG_KIND** | `nvarchar(4)` | NULL | 組織類型（冗餘，對應 SYS_ORGKIND.ORG_KIND） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。ROLE_ID 早期為 `varchar(20)`，後透過 ALTER TABLE 擴展為 `varchar(50)`。

---

## 主鍵

```
PRIMARY KEY (ORG_NO, ROLE_ID)
```

### 索引

```
INDEX ORGNO ON SYS_ORGROLES (ORG_NO, ROLE_ID)
```

---

## 資料生命週期

```
新增（角色→組織視角）：Board.cshtml → ucRoleOrg
      → ucRoleOrg_onBeforeInsert()：INSERT INTO SYS_ORGROLES (ORG_NO, ROLE_ID, ORG_KIND)
      → roleOrg_onBeforeExecuteSQL()：WHERE GROUPS.ISROLE = 'Y' 過濾只顯示角色

刪除：ucRoleOrg_onBeforeApply()
      → DELETE FROM SYS_ORGROLES WHERE ORG_NO = @ORG_NO AND ROLE_ID = @ROLE_ID

登入載入：AccountProvider.GetOrgRoles(db, roles, orgKind)
      → SELECT ORG_NO FROM SYS_ORG WHERE ORG_MAN IN (@roles) AND ORG_KIND = @orgKind
      → 遞迴取下屬組織 GetOrgs()
      → SELECT ROLE_ID FROM SYS_ORGROLES WHERE ORG_NO IN (@orgNos) AND ORG_KIND = @orgKind
      → 合併到 ClientInfo.OrgRoles
```

---

## 備註

- ROLE_ID 必須是 GROUPS 中 ISROLE='Y' 的角色。`roleOrg_onBeforeExecuteSQL` 強制加入 `WHERE GROUPS.ISROLE = 'Y'` 過濾。
- ORG_KIND 為冗餘欄位（可從 SYS_ORG.ORG_KIND 取得），方便查詢時不需額外 JOIN。
- 組織樹 Runtime 模式中，SYS_ORGROLES 的角色以 `'R_' + ROLE_ID` 作為虛擬 ORG_NO，UPPER_ORG 指向實際的 ORG_NO，形成組織→角色的父子關係。
