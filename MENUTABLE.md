# MENUTABLE

## 用途

**選單主檔**（Menu Master）。

MENUTABLE 是 EEP Core 選單系統的核心表，儲存所有選單項目的定義，包含階層結構（樹狀）、頁面路徑、所屬方案、版本控制、多語系標題等。整個 EEP 的功能入口都由此表驅動。

### 使用場景

| 場景 | 說明 |
|------|------|
| **選單載入（Runtime）** | `runtimeMenu` infocommand + `runtimeMenu_onBeforeExecuteSQL()`：以 ITEMTYPE + 權限（USERMENUS ∪ GROUPMENUS）過濾，按 SEQ_NO + MENUID 排序 |
| **選單樹渲染** | `runtimeMenuTable` infocommand：只取 MENUID/CAPTION/PARENT/FORM/ITEMTYPE/SEQ_NO，前端根據 PARENT 建立樹狀結構 |
| **選單管理（設計端）** | `menu` infocommand：`SELECT * FROM MENUTABLE ORDER BY SEQ_NO, MENUID`，以 ITEMTYPE 過濾當前方案 |
| **新增選單** | `ucMenu_onBeforeInsert()`：自動產生 MENUID（MAX+1）、填入 ITEMTYPE、自動為 EveryOne 群組授權 |
| **權限控制** | GROUPMENUS / USERMENUS 根據 MENUID 控制存取權限 |
| **版本控制** | VERSIONNO / CHECKOUT / CHECKOUTDATE 支援選單的版本管理與鎖定 |
| **多語系** | CAPTION0~7 支援 8 種語系的選單標題 |
| **我的最愛** | MENUFAVOR LEFT JOIN MENUTABLE 取得 IMAGEURL/ITEMPARAM/MODULETYPE/FORM |

### 選單樹結構

```
MENUID='0'（根節點，PARENT 為空）
├── MENUID='1'（CAPTION='系統管理', PARENT='0'）
│   ├── MENUID='101'（CAPTION='使用者管理', PARENT='1'）
│   └── MENUID='102'（CAPTION='權限管理', PARENT='1'）
└── MENUID='2'（CAPTION='業務模組', PARENT='0'）
```

### 關聯表

```
MENUTABLE ──(ITEMTYPE)────> MENUITEMTYPE        （方案定義）
          ──(MENUID)──────< GROUPMENUS          （群組選單權限，一對多）
          ──(MENUID)──────< USERMENUS           （使用者選單權限，一對多）
          ──(MENUID)──────< MENUTABLECONTROL    （控制項定義，一對多）
          ──(MENUID)──────< MENUFAVOR           （我的最愛，一對多）
          ──(PARENT)──────> MENUTABLE           （上層選單，自我關聯）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | `runtimeMenu_onBeforeExecuteSQL()`：Runtime 選單過濾（ITEMTYPE + 權限）；`runtimeMenuTable_onBeforeExecuteSQL()`：簡化版 |
| `SystemTable.Core/UserModule.cs` | `ucMenu_onBeforeInsert()`：新增選單自動產生 MENUID 並授權 EveryOne；`menu_onBeforeExecuteSQL()`：設計端以 ITEMTYPE 過濾 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `ExportUserAccess()`/`ExportGroupUser()`/`ExportTreeNode()`：權限匯出，JOIN MENUTABLE 取 CAPTION |
| `EEPWebClient.Core/design/server/SystemTable.json` | `menu`：設計端選單管理；`runtimeMenu`：Runtime 選單載入；`runtimeMenuTable`：簡化版選單樹 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **MENUID** | `nvarchar(30)` PK | NOT NULL | 選單識別碼（MAX+1 自動產生） |
| **CAPTION** | `nvarchar(50)` | NULL | 選單顯示標題 |
| **PARENT** | `nvarchar(20)` | NULL | 父選單 MENUID（空值表示根節點） |
| **PACKAGE** | `nvarchar(60)` | NULL | 頁面路徑/組件名稱 |
| **MODULETYPE** | `nvarchar(1)` | NULL | 模組類型 |
| **ITEMPARAM** | `nvarchar(200)` | NULL | 選單參數（開啟頁面時傳遞） |
| **FORM** | `nvarchar(32)` | NULL | 表單識別碼 |
| **ISSHOWMODAL** | `nvarchar(1)` | NULL | 是否以對話框模式開啟（`Y`/`N`） |
| **ITEMTYPE** | `nvarchar(20)` | NULL | 所屬方案（對應 MENUITEMTYPE.ITEMTYPE） |
| **SEQ_NO** | `nvarchar(4)` | NULL | 排序序號 |
| **PACKAGEDATE** | `datetime` | NULL | 組件打包日期 |
| **IMAGE** | `image` | NULL | 選單圖示（二進位） |
| **OWNER** | `nvarchar(20)` | NULL | 擁有者 USERID |
| **ISSERVER** | `nvarchar(1)` | NULL | 是否為伺服器端組件 |
| **VERSIONNO** | `nvarchar(20)` | NULL | 版本號 |
| **CHECKOUT** | `nvarchar(20)` | NULL | 簽出者 USERID（版本鎖定） |
| **CHECKOUTDATE** | `datetime` | NULL | 簽出日期 |
| **CAPTION0~7** | `nvarchar(50)` | NULL | 語系 0~7 標題（8 個欄位） |
| **IMAGEURL** | `nvarchar(100)` | NULL | 選單圖示 URL（替代 IMAGE 二進位） |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| IMAGE | `image` | `BLOB` | `BLOB` | `BLOB (100M)` |
| PACKAGEDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |
| CHECKOUTDATE | `datetime` | `date` | `datetime` | `DATETIME YEAR TO SECOND` |

---

## 主鍵

```
PRIMARY KEY (MENUID)
```

---

## 預設資料

```sql
INSERT INTO MENUTABLE(MENUID, CAPTION, MODULETYPE, ITEMTYPE) VALUES('0', N'iCoder系統', 'B', 'SOLUTION1');
```

---

## 資料生命週期

```
新增選單：設計端 → ucMenu_onBeforeInsert()
      → MENUID = MAX(MENUID) + 1，ITEMTYPE = ClientInfo.Solution
      → 自動為 EveryOne 群組授權：DELETE + INSERT INTO GROUPMENUS ('00', @MENUID)

Runtime 載入：使用者登入後
      → runtimeMenu_onBeforeExecuteSQL()
      → SELECT * FROM MENUTABLE
        WHERE ITEMTYPE = @Solution
        AND (MENUID IN (SELECT MENUID FROM USERMENUS WHERE USERID = @User)
         OR  MENUID IN (SELECT MENUID FROM GROUPMENUS WHERE GROUPID IN (@Groups)))
        ORDER BY SEQ_NO, MENUID
```

---

## 備註

- MENUID='0' 是根節點，所有頂層選單的 PARENT 指向它。預設 CAPTION 為 'iCoder系統'。
- MENUID 的產生方式為 MAX(MENUID)+1，非自動遞增。
- IMAGE 和 IMAGEURL 二擇一：IMAGE 儲存二進位圖示，IMAGEURL 提供 URL 替代方案。
- 版本控制機制（CHECKOUT/CHECKOUTDATE）防止多人同時修改同一選單項目。
- `runtimeMenuTable` 是 `runtimeMenu` 的精簡版，只取 6 個核心欄位，用於建構選單樹。
