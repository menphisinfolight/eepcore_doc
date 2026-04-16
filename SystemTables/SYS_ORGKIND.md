# SYS_ORGKIND

## 用途

**組織類型定義表**（Organization Kind Definition）。

SYS_ORGKIND 定義組織單位的分類類型，如公司組織、事業群等。SYS_ORG.ORG_KIND 關聯此表，流程引擎的主管查詢和組織樹查詢都以 ORG_KIND 過濾。

### 使用場景

| 場景 | 說明 |
|------|------|
| **組織過濾** | `AccountProvider.GetOrgRoles()` 和 `FlowProvider.GetManagerID()` 查詢 SYS_ORG 時以 ORG_KIND 過濾 |
| **組織樹 Runtime** | `org_onBeforeExecuteSQL()` Runtime 模式下的 WHERE 條件含 ORG_KIND |
| **組織類型管理** | `orgKind` infocommand 提供組織類型的 CRUD |
| **冗餘欄位** | SYS_ORGROLES 也有 ORG_KIND 欄位（冗餘），方便查詢時不需 JOIN SYS_ORG |

### 關聯表

```
SYS_ORGKIND ──(ORG_KIND)──< SYS_ORG       （組織架構，一對多）
            ──(ORG_KIND)──< SYS_ORGROLES  （組織角色，冗餘欄位）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `GetOrgRoles()`/`GetOrgs()`：以 ORG_KIND 過濾查詢 |
| `EEPServerTools.Core/Adapter/Flow.cs` | `GetManagerID()`：ORG_KIND 預設 '0'，查詢主管時過濾 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `orgKind` infocommand：`SELECT * FROM SYS_ORGKIND ORDER BY ORG_KIND` |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ORG_KIND** | `nvarchar(4)` PK | NOT NULL | 組織類型代碼 |
| **KIND_DESC** | `nvarchar(40)` | NOT NULL | 類型描述 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (ORG_KIND)
```

---

## 預設資料

```sql
INSERT INTO SYS_ORGKIND(ORG_KIND, KIND_DESC) VALUES ('0', N'公司組織');
```

---

## 備註

- 預設只有一種組織類型 '0'（公司組織），可依需求新增其他類型（如事業群、子公司等）。
- FlowProvider.GetManagerID() 中 orgKind 預設為 '0'，若未傳入則使用預設值。
- ClientInfo.OrgKind 在登入時設定，決定使用者使用哪種組織體系。
