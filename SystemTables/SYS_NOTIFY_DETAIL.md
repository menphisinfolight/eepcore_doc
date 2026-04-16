# SYS_NOTIFY_DETAIL

## 用途

**系統公告接收者明細表**（Notification Recipient Detail）。

SYS_NOTIFY_DETAIL 定義每則系統公告的接收者清單，與 SYS_NOTIFY 為多對一關係。設計用途是記錄哪些使用者已點擊「不再顯示」或作為公告的發送目標清單。

> ⚠️ **未實際使用**：此表在 SQL 建表腳本中有定義，但 C# 程式碼和前端 JavaScript 均無引用。NotifyProvider 的 GetNotifies() 和 DeleteNotify() 尚未實作讀寫 SYS_NOTIFY_DETAIL 的邏輯。

### 關聯表

```
SYS_NOTIFY_DETAIL ──(NotifyID)──> SYS_NOTIFY  （系統公告）
                  ──(UserID)───> USERS        （使用者）
```

### 預期使用場景（根據表結構推斷）

| 場景 | 說明 |
|------|------|
| **記錄已讀/不再顯示** | 使用者點擊「不再顯示」時 INSERT (NotifyID, UserID)，下次載入時排除已記錄的公告 |
| **指定發送對象** | 或者作為公告的目標使用者清單，僅發送給指定使用者 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **NotifyID** | `int` PK | NOT NULL | 公告識別碼（對應 SYS_NOTIFY.ID） |
| **UserID** | `varchar(20)` PK | NOT NULL | 使用者帳號（對應 USERS.USERID） |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 / Informix |
|------|-----------|-------|--------|----------------|
| NotifyID | `int` | `int` | `integer` | `INT` / `INTEGER` |
| UserID | `varchar(20)` | `varchar(20)` | `varchar2(20)` | `VARCHAR(20)` / `NVARCHAR(20)` |

---

## 主鍵

```
PRIMARY KEY (NotifyID, UserID)
```

---

## 備註

- 此表有完整的建表腳本（SQL Server / MySQL / Oracle / DB2 / Informix 全版本），但無任何 infocommand 定義、無 C# Provider 操作、無前端引用。
- 推測為 SYS_NOTIFY 功能的配套表，待 NotifyProvider 實作完成後啟用。
- NotifyProvider.GetNotifies() 實作時，預期會以 `SYS_NOTIFY LEFT JOIN SYS_NOTIFY_DETAIL` 排除已標記「不再顯示」的使用者。
- NotifyProvider.DeleteNotify() 實作時，預期會 INSERT 一筆 (NotifyID, UserID) 記錄。
