# USERMENUCONTROL

## 用途

**使用者選單控制項權限表**（User Menu Control Permissions）。

USERMENUCONTROL 定義個別使用者在特定選單頁面上，對個別 UI 控制項（按鈕、欄位等）的細緻權限。與 GROUPMENUCONTROL 為對偶表，結構完全相同，僅主鍵從 GROUPID 改為 USERID。

> ⚠️ **程式碼無引用**：此表在 SQL 建表腳本中有完整定義，但 EEPCore SP7 的 C# 程式碼、前端 JavaScript 和 SystemTable JSON 均無引用。無對應的 infocommand、無 UpdateComponent、無 Provider 操作。僅在系統表列表 `jquery.infolight.table.system.json` 中出現。推測為預留結構或由其他模組使用。

### 權限層級關係

```
USERMENUS（頁面級）：ALLOWADD / ALLOWUPDATE / ALLOWDELETE
    └─ USERMENUCONTROL（控制項級）：ENABLED / VISIBLE / ALLOWADD / ALLOWUPDATE / ALLOWDELETE / ALLOWPRINT
```

### 關聯表

```
USERMENUCONTROL ──(USERID)──────────> USERS      （使用者帳號）
                ──(MENUID)──────────> MENUTABLE  （選單定義）
                ──(CONTROLNAME)────> 前端 UI 元件
```

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **MENUID** | `varchar(30)` PK | NOT NULL | 選單識別碼（對應 MENUTABLE.MENUID） |
| **CONTROLNAME** | `varchar(50)` PK | NOT NULL | 控制項名稱（前端元件的 ID/Name） |
| **TYPE** | `varchar(20)` | NULL | 控制項類型（如 Button、Field） |
| **ENABLED** | `char(1)` | NULL | 是否啟用（`Y`/`N`） |
| **VISIBLE** | `char(1)` | NULL | 是否可見（`Y`/`N`） |
| **ALLOWADD** | `char(1)` | NULL | 此控制項是否允許新增操作 |
| **ALLOWUPDATE** | `char(1)` | NULL | 此控制項是否允許修改操作 |
| **ALLOWDELETE** | `char(1)` | NULL | 此控制項是否允許刪除操作 |
| **ALLOWPRINT** | `char(1)` | NULL | 此控制項是否允許列印操作 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar`。無特殊型別差異。

---

## 主鍵

```
PRIMARY KEY (USERID, MENUID, CONTROLNAME)
```

---

## 備註

- 與 GROUPMENUCONTROL 為對偶表，結構完全相同，僅主鍵不同。
- 預期使用者最終控制項權限 = USERMENUCONTROL ∪ GROUPMENUCONTROL（透過使用者所屬群組），但目前程式碼中未找到此合併邏輯。
- GROUPMENUCONTROL 同樣在 C# 和 JS 中無引用（僅在系統表列表中出現），兩者均為預留結構。
