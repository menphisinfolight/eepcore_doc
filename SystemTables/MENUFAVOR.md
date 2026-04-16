# MENUFAVOR

## 用途

**選單我的最愛表**（Menu Favorites）。

MENUFAVOR 儲存使用者的選單收藏/最愛設定，讓使用者可以快速存取常用功能。前端提供我的最愛管理介面，勾選/取消選單即時儲存。

### 使用場景

| 場景 | 說明 |
|------|------|
| **我的最愛載入** | `runtimeMenuFavor` infocommand LEFT JOIN MENUTABLE 取得完整選單資訊（含 IMAGEURL、ITEMPARAM 等），以 USERID + ITEMTYPE 過濾 |
| **我的最愛管理** | 前端 `checkFavor()` 開啟 modalFavor 對話框，勾選/取消後透過 `saveMenusFavor` 批次儲存 |
| **防重複 INSERT** | `ucMenuFavor_onBeforeInsert()` 在 INSERT 前先 DELETE 同一 MENUID + USERID 的記錄，確保不重複 |
| **方案重新命名** | `SecurityProvider.RenameSolution()` 同步更新 MENUFAVOR 的 ITEMTYPE |
| **選單面板分組** | 前端以 GROUPNAME 將收藏項目分組顯示於側邊欄 |

### 關聯表

```
MENUFAVOR ──(MENUID + ITEMTYPE)──> MENUTABLE   （選單定義，LEFT JOIN 取得 IMAGEURL 等）
          ──(USERID)─────────────> USERS       （使用者帳號）
          ──(ITEMTYPE)───────────> MENUITEMTYPE （方案）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/DataProvider.cs` | `saveMenusFavor`：批次儲存（自動填入 USERID + ITEMTYPE，inserted + deleted） |
| `SystemTable.Core/UserModule.cs` | `runtimeMenuFavor_onBeforeExecuteSQL()`：加入 USERID + ITEMTYPE 過濾條件 |
| `SystemTable.Core/UserModule.cs` | `ucMenuFavor_onBeforeInsert()`：INSERT 前先 DELETE 防重複 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `RenameSolution()`：方案重新命名時同步 UPDATE ITEMTYPE |
| `EEPWebClient.Core/design/server/SystemTable.json` | `runtimeMenuFavor`：LEFT JOIN MENUTABLE 取完整選單資訊；`menuFavor`：基本 CRUD |
| `EEPWebClient.Core/wwwroot/js/infolight/bootstrap.infolight.main.js` | `checkFavor()`：開啟我的最愛管理對話框，儲存透過 DataProvider.saveMenusFavor |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **MENUID** | `nvarchar(30)` PK | NOT NULL | 選單識別碼（對應 MENUTABLE.MENUID） |
| **USERID** | `varchar(20)` PK | NULL（建表時）| 使用者帳號（saveMenusFavor 時自動填入 ClientInfo.User） |
| **CAPTION** | `nvarchar(50)` | NOT NULL | 選單標題（快照，避免每次 JOIN 查詢） |
| **ITEMTYPE** | `nvarchar(20)` | NULL | 所屬方案（saveMenusFavor 時自動填入 ClientInfo.Solution） |
| **GROUPNAME** | `nvarchar(20)` | NULL | 群組名稱（用於分類最愛項目） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar/nvarchar`。無特殊型別差異。

---

## 主鍵

```
PRIMARY KEY (MENUID, USERID)
```

---

## 資料生命週期

```
新增收藏：前端 checkFavor() → 勾選選單
      → DataProvider.saveMenusFavor
      → 自動填入 USERID = ClientInfo.User, ITEMTYPE = ClientInfo.Solution
      → ucMenuFavor_onBeforeInsert() → DELETE 同鍵記錄 → INSERT

取消收藏：前端 checkFavor() → 取消勾選
      → DataProvider.saveMenusFavor → deleted 陣列
      → DELETE FROM MENUFAVOR WHERE MENUID = @MENUID AND USERID = @USERID

載入：使用者登入後顯示我的最愛
      → runtimeMenuFavor infocommand
      → SELECT MENUFAVOR.*, IMAGEURL, ITEMPARAM, MODULETYPE, FORM
        FROM MENUFAVOR LEFT JOIN MENUTABLE
        ON MENUFAVOR.MENUID = MENUTABLE.MENUID AND MENUFAVOR.ITEMTYPE = MENUTABLE.ITEMTYPE
        WHERE USERID = @User AND MENUFAVOR.ITEMTYPE = @Solution
        ORDER BY MENUFAVOR.MENUID
```

---

## 備註

- CAPTION 是冗餘快照欄位，寫入時取自 MENUTABLE.CAPTION，但 MENUTABLE 的 CAPTION 變更時不會自動同步。
- USERID 和 ITEMTYPE 在前端不需傳入，由 DataProvider.saveMenusFavor 自動從 ClientInfo 填入。
- `ucMenuFavor_onBeforeInsert` 採「先刪後寫」策略（return true 表示仍執行預設 INSERT），確保不產生重複記錄。
- `runtimeMenuFavor` LEFT JOIN MENUTABLE 可取得 IMAGEURL（圖示）、ITEMPARAM（參數）、MODULETYPE（模組類型）、FORM（表單名稱）等顯示所需欄位。
- 前端選單面板中，收藏項目以 `<--root-->` 以外的 GROUPNAME 分組顯示。
