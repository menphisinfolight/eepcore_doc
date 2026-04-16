# MENUTABLECONTROL

## 用途

**選單控制項定義表**（Menu Control Definitions）。

MENUTABLECONTROL 定義每個選單頁面上可用的 UI 控制項清單，是 GROUPMENUCONTROL / USERMENUCONTROL 設定權限的基礎。系統管理者先在此表定義頁面有哪些控制項，再透過權限表決定各群組/使用者對這些控制項的啟用/可見權限。

> ⚠️ **C# 程式碼無直接引用**：此表在 SQL 建表腳本中有完整定義��但 C# 和 JS 中未找到直接引用 `MENUTABLECONTROL` 的操作。與 GROUPMENUCONTROL / USERMENUCONTROL 一樣，推測為預留結構或由 DataModule 通用框架動態操作。

### 權限架構關係

```
MENUTABLECONTROL（定義頁面有哪些控制項）
    ↓ 權限設定基於此
GROUPMENUCONTROL（群組對控制項的 ENABLED/VISIBLE 等權限）
USERMENUCONTROL（使用者對控制項的 ENABLED/VISIBLE 等權限）
```

### 關聯表

```
MENUTABLECONTROL ──(MENUID)──> MENUTABLE          （選單定義）
                 ──(MENUID + CONTROLNAME)──< GROUPMENUCONTROL  （群組權限）
                 ──(MENUID + CONTROLNAME)──< USERMENUCONTROL   （使用者權限）
```

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **MENUID** | `varchar(30)` PK | NOT NULL | 選單識別碼（對應 MENUTABLE.MENUID） |
| **CONTROLNAME** | `varchar(50)` PK | NOT NULL | 控制項名稱（前端元件的 ID/Name） |
| **DESCRIPTION** | `varchar(50)` | NULL | 控制項描述 |
| **TYPE** | `varchar(20)` | NULL | 控制項類型（如 Button、Field、Toolbar） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (MENUID, CONTROLNAME)
```

---

## 備註

- 此表是控制項的「定義檔」，GROUPMENUCONTROL / USERMENUCONTROL 是控制項的「權限檔」。
- 三者的 CONTROLNAME 必須一致，權限設定才會生效。
- 僅在系統表列表 `jquery.infolight.table.system.json` 中出現。
- GROUPMENUCONTROL 和 USERMENUCONTROL 在程式碼中也無引用，整組控制項權限機制可能為預留功能。
