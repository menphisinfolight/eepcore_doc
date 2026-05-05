# SYS_ORG

## 用途

**組織架構主檔**（Organization Structure Master）。

SYS_ORG 儲存公司的組織架構，包含部門/單位的階層關係、主管指派、組織層級等。支援樹狀組織結構，用於流程簽核的主管路由（GetManagerID）、組織權限範圍控制等。

### 使用場景

| 場景 | 說明 |
|------|------|
| **登入載入 OrgRoles** | `AccountProvider.GetOrgRoles()` 根據使用者的 Roles 查詢 SYS_ORG 找到管轄的組織，再遞迴取得所有下屬組織的 ORG_MAN 和 SYS_ORGROLES |
| **流程主管簽核** | `FlowProvider.GetManagerID()` 根據 ROLE_ID 查 SYS_ORGROLES → SYS_ORG，找到上層組織的 ORG_MAN（主管角色） |
| **組織樹顯示** | `org` infocommand UNION SYS_ORG（TYPE='R'）+ SYS_ORGROLES JOIN GROUPS（TYPE='U'），合併顯示組織單位和角色 |
| **組織管理** | `ucOrg_onBeforeInsert()` 自動產生 ORG_NO（MAX+1） |
| **下屬組織遞迴** | `AccountProvider.GetOrgs()` 以 UPPER_ORG 遞迴查詢所有下屬組織 |

### 組織樹結構

```
ORG_NO='1'（總公司，UPPER_ORG 為空）
├── ORG_NO='101'（資訊部，UPPER_ORG='1'）
│   └── ORG_NO='1011'（應用組，UPPER_ORG='101'）
└── ORG_NO='102'（業務部，UPPER_ORG='1'）
```

### 關聯表

```
SYS_ORG ──(ORG_KIND)──> SYS_ORGKIND   （組織類型）
        ──(LEVEL_NO)──> SYS_ORGLEVEL  （組織層級）
        ──(ORG_MAN)───> GROUPS        （主管角色，透過 USERGROUPS 找到使用者）
        ──(ORG_NO)────> SYS_ORGROLES  （組織角色，一對多）
        ──(UPPER_ORG)─> SYS_ORG       （上層組織，自我關聯）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `GetOrgRoles()`：登入時查詢管轄組織及下屬角色；`GetOrgs()`：遞迴查下屬組織 |
| `EEPServerTools.Core/Adapter/Flow.cs` | `FlowProvider.GetManagerID()`：流程簽核時查詢上層主管 |
| `SystemTable.Core/UserModule.cs` | `org_onBeforeExecuteSQL()`：Runtime 模式 UNION SYS_ORG + SYS_ORGROLES；`ucOrg_onBeforeInsert()`：自動產生 ORG_NO |
| `EEPWebClient.Core/design/server/SystemTable.json` | `org` infocommand：含 USERNAME 子查詢；`orgRole`：組織→角色明細 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportOrg()`：匯出組織清單時用遞迴 CTE / `CONNECT BY` 重算 `ORG_TREE` 並 `ORDER BY` 它做樹狀排序（**唯一使用 `ORG_TREE` 欄位的功能**） |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ORG_NO** | `nvarchar(8)` PK | NOT NULL | 組織編號（MAX+1 自動產生） |
| **ORG_DESC** | `nvarchar(40)` | NOT NULL | 組織名稱 |
| **ORG_KIND** | `nvarchar(4)` | NOT NULL | 組織類型代碼（對應 SYS_ORGKIND.ORG_KIND） |
| **UPPER_ORG** | `nvarchar(8)` | NULL | 上層組織編號（空值表示根節點） |
| **ORG_MAN** | `nvarchar(50)` | NOT NULL | 部門主管（GROUPID，非 USERID） |
| **LEVEL_NO** | `nvarchar(6)` | NOT NULL | 組織層級代碼（對應 SYS_ORGLEVEL.LEVEL_NO） |
| **ORG_TREE** | `nvarchar(40)` | NULL | **衍生排序鍵**（非業務資料）。僅匯出組織清單時即時重算+排序用，平常多為 NULL。格式為串接的兄弟序號（如 `010203`），不是路徑 |
| **END_ORG** | `nvarchar(4)` | NULL | 是否為末梢組織（`Y`/`N`） |
| **ORG_FULLNAME** | `nvarchar(254)` | NULL | 組織全名（含上層路徑） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `nvarchar`。

---

## 主鍵

```
PRIMARY KEY (ORG_NO)
```

---

## 預設資料

```sql
INSERT INTO SYS_ORG(ORG_NO, ORG_DESC, ORG_KIND, ORG_MAN, LEVEL_NO) VALUES ('1', N'總公司', '0', '001', '9');
```

---

## 流程主管查詢邏輯（GetManagerID）

```
1. 以 ROLE_ID 查 SYS_ORGROLES 找到所屬 ORG_NO
2. 同時查 SYS_ORG 中 ORG_MAN = ROLE_ID 的 UPPER_ORG
3. 取得 ORG_MAN 和 LEVEL_NO
4. 若 levelNo 為空/'0' 或與當前 LEVEL_NO 相符 → 回傳 ORG_MAN
5. 若不符 → 以 UPPER_ORG 遞迴向上查詢（GetManagerID(roleID, upperOrg, levelNo, orgKind)）
```

---

## 備註

- ORG_MAN 存的是 GROUPID（角色識別碼），非 USERID。需透過 USERGROUPS 找到實際使用者。
- ORG_NO 的產生方式為 MAX(ORG_NO)+1，非自動遞增。
- `org_onBeforeExecuteSQL` 在 Runtime 模式下以 UNION 合併 SYS_ORG（TYPE='R'）和 SYS_ORGROLES JOIN GROUPS（TYPE='U'），讓前端組織樹同時顯示組織單位和角色。
- OrgKind 預設為 '0'（公司組織），GetManagerID 的所有查詢都以 ORG_KIND 過濾。
- **ORG_TREE 是衍生欄位、不是業務邏輯依賴的資料**：只在 `SecurityProvider.ExportOrg()`（匯出組織清單）時即時重算+排序使用。組織樹的正常載入（`org` infocommand、`AccountProvider.GetOrgs()`、`FlowProvider.GetManagerID()`）都是以 `UPPER_ORG` 父子關係即時遞迴建出，與 `ORG_TREE` 欄位無關。前端 JS 中的 `onOrgTreeBeforeLoad` / `onOrgTreeLoadError` 是組織樹 UI 元件的事件 callback，名稱巧合，與此欄位無關。
