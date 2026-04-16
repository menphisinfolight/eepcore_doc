# SYS_ANYQUERY

> ⚠️ **狀態**：舊版遺留，**未實際使用**
>
> 此表雖在 SQL 腳本中有定義，但 C# 與 JavaScript 程式碼均無引用。

---

## 用途

**自訂查詢儲存表**（Any Query / Saved Query Templates）。

SYS_ANYQUERY 儲存使用者自訂的查詢條件組合，讓使用者可以儲存常用的查詢設定，下次使用時直接載入，不需重新設定查詢條件。

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **QUERYID** | `varchar(20)` PK | 查詢識別碼 |
| **USERID** | `varchar(20)` PK | 使用者帳號（對應 USERS.USERID） |
| **TEMPLATEID** | `varchar(20)` PK | 查詢模板識別碼（區分同一查詢的不同版本） |
| **TABLENAME** | `varchar(50)` | 查詢目標資料表名稱 |
| **LASTDATE** | `datetime` | 最後使用日期 |
| **CONTENT** | `text` | 查詢條件內容（序列化的查詢設定） |

---

## 主鍵

```
PRIMARY KEY (QUERYID, USERID, TEMPLATEID)
```

---

## 備註

- CONTENT 使用 `text` 類型儲存序列化的查詢設定（可能是 JSON 或 XML）。
- 三欄位複合主鍵確保同一使用者的同一查詢可有多個模板版本。
