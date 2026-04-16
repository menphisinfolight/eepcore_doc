# MENUTABLELOG

> ⚠️ **狀態**：舊版遺留，**未實際使用**
>
> 此表雖在 SQL 腳本中有定義，但 C# 與 JavaScript 程式碼均無引用（MENUTABLE 有用，但 MENUTABLELOG 無）。

---

## 用途

**選單版本變更記錄表**（Menu Version Change Log）。

MENUTABLELOG 記錄選單項目的版本變更歷史，包含舊版本的二進位內容（OLDVERSION），用於版本追溯與回滾。

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **LOGID** | `int identity(1,1)` PK | 自動遞增記錄識別碼 |
| **MENUID** | `nvarchar(30)` | 選單識別碼（對應 MENUTABLE.MENUID） |
| **PACKAGE** | `nvarchar(20)` | 組件名稱 |
| **PACKAGEDATE** | `datetime` | 組件打包日期 |
| **LASTDATE** | `datetime` | 最後修改日期 |
| **OWNER** | `nvarchar(20)` | 修改者 USERID |
| **OLDVERSION** | `image` | 舊版本內容（二進位） |
| **OLDDATE** | `nvarchar(20)` | 舊版本日期 |

---

## 主鍵

```
PRIMARY KEY (LOGID)
```

---

## 備註

- OLDVERSION 使用 `image` 類型儲存整個舊版本的二進位快照，可用於版本回滾。
- 與 MENUTABLE 的版本控制機制（CHECKOUT/VERSIONNO）搭配使用。
