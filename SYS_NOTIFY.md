# SYS_NOTIFY

## 用途

**系統公告表**（System Notification / Announcement）。

SYS_NOTIFY 儲存系統公告/通知，管理者可在設計端新增公告並設定有效期。公告會在使用者登入時以 messager 訊息框顯示，使用者可點擊「不再顯示」關閉。

### 使用場景

| 場景 | 說明 |
|------|------|
| **公告管理（設計端）** | Notify.cshtml 提供新增/編輯/刪除/重新整理功能，透過 `notify` infocommand 操作 SYS_NOTIFY |
| **公告顯示（前端）** | `$.fn.notify.load()` 透過 NotifyProvider 載入公告，以 `$.messager.show()` 顯示 |
| **不再顯示** | `$.fn.notify.delete(id)` 透過 NotifyProvider.DeleteNotify() 標記不再顯示 |
| **ID 自動產生** | `ucNotify_onBeforeInsert()` 查詢 `SELECT ID FROM SYS_NOTIFY` 取得 MAX(ID)+1 作為新 ID |

> ⚠️ **NotifyProvider 未完整實作**：`GetNotifies()` 目前回傳 `"[]"`（空陣列），`DeleteNotify()` 回傳空字串。前端程式碼已就緒但後端功能尚未完成。SYS_NOTIFY_DETAIL 的讀寫邏輯也未在 Provider 中實作。

### 關聯表

```
SYS_NOTIFY ──(ID)──> SYS_NOTIFY_DETAIL  （接收者明細，一對多）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/NotifyProvider.cs` | API：GetNotifies()（⚠️ 回傳空陣列）、DeleteNotify()（⚠️ 回傳空字串） |
| `SystemTable.Core/UserModule.cs` | `ucNotify_onBeforeInsert()`：自動產生 ID（MAX+1）、填入 Sender 和 Datetime |
| `EEPWebClient.Core/design/server/SystemTable.json` | `notify` infocommand：`SELECT * FROM SYS_NOTIFY`，keys: `ID` |
| `EEPWebClient.Core/Views/Design/Notify.cshtml` | 前端公告管理介面（gridNotify + editNotify 表單） |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.design.js` | `$.fn.notify.load()` / `$.fn.notify.delete(id)`：前端載入/關閉公告 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **ID** | `int` PK | NOT NULL | 公告識別碼（非自動遞增，由 ucNotify_onBeforeInsert 產生 MAX+1） |
| **Text** | `nvarchar(max)` | NULL | 公告內容 |
| **Sender** | `varchar(20)` | NULL | 發送者帳號（自動填入當前使用者） |
| **Datetime** | `datetime` | NULL | 發送時間（自動填入 `yyyy-MM-dd hh:mm:ss`） |
| **ValidateDate** | `datetime` | NULL | 有效期截止日（前端以 datebox 選擇） |

### 跨資料庫差異

| 欄位 | SQL Server | MySQL | Oracle | DB2 / Informix |
|------|-----------|-------|--------|----------------|
| Text | `nvarchar(max)` | `text` | `clob` | `CLOB (100M)` / `LVARCHAR(9800)` |
| Datetime | `datetime` | `datetime` | `date`（欄位名為 `"date"`） | `DATETIME YEAR TO SECOND` |
| ValidateDate | `datetime` | `datetime` | `date` | `DATETIME YEAR TO SECOND` |

> ⚠️ Oracle 版的 Datetime 欄位名為 `"date"`（帶引號），與其他資料庫的 `Datetime` 不同。

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 前端顯示邏輯

```javascript
// 登入後載入公告
$.fn.notify.load()
  → POST { type: 'notify', mode: 'load' }
  → NotifyProvider.GetNotifies() → 目前回傳 "[]"

// 顯示為 messager（每則公告一個訊息框）
$.messager.show({
  title: '系統公告',
  msg: row.Text + '不再顯示連結',
  timeout: 0  // 不自動關閉
})

// 點擊「不再顯示」
$.fn.notify.delete(id)
  → POST { type: 'notify', mode: 'remove', id: id }
  → NotifyProvider.DeleteNotify() → 目前回傳 ""
```

---

## 備註

- ID 的產生方式為查詢 MAX(ID)+1，非使用資料庫自動遞增，可能在高並發下有衝突風險。
- Sender 和 Datetime 在 `ucNotify_onBeforeInsert` 中自動填入，前端不需傳入。
- NotifyProvider 是 BaseProvider 中 `case "notify"` 的處理類別。
- 前端 `$.fn.notify.formatDatetime` 將日期格式化為只顯示日期部分（去掉時間）。
- 卡片系統（systeminfocard.js）中 CARDTYPE = 'notify' 會呼叫 `$.fn.flow.renderFlowCard('notify')`，這是流程通知卡片，與 SYS_NOTIFY 系統公告無關。
