# MENUITEMTYPE

## 用途

**選單方案定義表**（Menu Item Type / Solution Definition）。

MENUITEMTYPE 定義系統中的「方案」（Solution），每個方案代表一組選單集合。EEP Core 支援多方案切換，使用者可透過 SolutionProvider 切換當前方案，不同方案可看到不同的選單樹。

### 使用場景

| 場景 | 說明 |
|------|------|
| **方案列表查詢** | `SolutionProvider.GetSolutions()` 以 `db.SelectTable("MENUITEMTYPE")` 查詢所有方案，標記當前選中的 |
| **方案切換** | `SolutionProvider.SetSolution()` 將 ITEMTYPE 寫入 Session 和 Cookie（`devsolution`，7 天有效） |
| **選單過濾** | MENUTABLE.ITEMTYPE 決定選單屬於哪個方案，`runtimeMenu_onBeforeExecuteSQL` 以 `WHERE ITEMTYPE = @Solution` 過濾 |
| **DB 別名** | DBALIAS 可為不同方案指定不同的資料庫連線 |
| **排程任務** | SYS_SCHEDULE 以 Solution 欄位關聯方案 |
| **方案重新命名** | `SecurityProvider.RenameSolution()` 同步更新 MENUITEMTYPE + MENUTABLE + MENUFAVOR 的 ITEMTYPE |

### 關聯表

```
MENUITEMTYPE ──(ITEMTYPE)──< MENUTABLE   （選單定義，一對多）
             ──(ITEMTYPE)──< MENUFAVOR   （使用者我的最愛）
             ──(Solution)──< SYS_SCHEDULE（排程任務）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/SolutionProvider.cs` | `GetSolutions()`：查詢方案列表；`SetSolution()`：切換方案（寫入 Session + Cookie） |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | `RenameSolution()`：方案重新命名，同步更新關聯表 |
| `SystemTable.Core/UserModule.cs` | `runtimeMenu_onBeforeExecuteSQL()`：以 ITEMTYPE 過濾選單 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ITEMTYPE** | `nvarchar(20)` PK | NOT NULL | 方案識別碼（如 `SOLUTION1`） |
| **ITEMNAME** | `nvarchar(20)` | NULL | 方案顯示名稱 |
| **DBALIAS** | `nvarchar(50)` | NULL | 資料庫別名（可為此方案指定不同的資料庫連線） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (ITEMTYPE)
```

---

## 預設資料

```sql
INSERT INTO MENUITEMTYPE(ITEMTYPE, ITEMNAME) VALUES('SOLUTION1', 'DEFAULT SOLUTION');
```

---

## 備註

- 方案切換時會寫入 Cookie `devsolution`（有效期 7 天），下次登入時自動恢復上次選擇的方案。
- ClientInfo.Solution 儲存當前方案的 ITEMTYPE，整個系統的選單過濾、排程、卡片等都以此為依據。
- DBALIAS 可讓不同方案連接不同的資料庫，實現多租戶或多環境的分離。
