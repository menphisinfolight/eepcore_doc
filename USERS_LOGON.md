# USERS_LOGON

## 用途

**使用者登入驗證記錄表**（User Logon Validation Record）。

USERS_LOGON 記錄使用者的登入嘗試時間，用於密碼驗證次數控制（防暴力登入）。當系統設定了 `pwdDate.ValidCount`（最大驗證次數）和 `ValidInterval`（計算區間秒數）時，系統會檢查該區間內的登入嘗試次數是否超過上限。

### 使用場景

| 場景 | 說明 |
|------|------|
| **登入次數檢查** | `CheckValidateCount()` 查詢指定區間內的登入記錄筆數，若超過 ValidCount 則拒絕登入 |
| **登入嘗試記錄** | `AddValidateCount()` 每次登入嘗試時 INSERT 一筆記錄（無論成功或失敗） |
| **登入成功清除** | `ClearValidateCount()` 登入成功後 DELETE 該使用者的所有記錄，重設計數 |

### 關聯表

```
USERS_LOGON ──(USERID)──> USERS  （使用者帳號）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `CheckValidateCount()`：查詢區間內記錄筆數，判斷是否超過上限 |
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `AddValidateCount()`：INSERT 登入嘗試記錄 |
| `EEPGlobal.Core/Provider/AccountProvider.cs` | `ClearValidateCount()`：DELETE 該使用者所有記錄 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **USERID** | `varchar(20)` | NOT NULL | 使用者帳號（對應 USERS.USERID） |
| **TIME** | `varchar(14)` | NOT NULL | 登入嘗試時間（格式：`yyyyMMddHHmmss`） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2` 取代 `varchar`。無特殊型別差異。

---

## 無主鍵

此表無主鍵定義。同一 USERID 可有多筆記錄（每次登入嘗試一筆）。

---

## 資料生命週期

```
寫入：每次登入嘗試（含失敗）
      → AddValidateCount()
      → INSERT INTO USERS_LOGON (USERID, TIME) VALUES (@user, @now)

檢查：登入前檢查
      → CheckValidateCount()
      → SELECT * FROM USERS_LOGON WHERE USERID = @user AND TIME >= @threshold
      → threshold = 當前時間 - ValidInterval 秒
      → 若筆數 >= ValidCount → 回傳 "pwdValidExceed" 拒絕登入

清除：登入成功後
      → ClearValidateCount()
      → DELETE FROM USERS_LOGON WHERE USERID = @user
```

---

## 驗證次數控制邏輯

依賴 config 中的 `pwdDate` 設定：

| 設定項 | 說明 |
|--------|------|
| `pwdDate.ValidCount` | 區間內允許的最大登入嘗試次數 |
| `pwdDate.ValidInterval` | 計算區間（秒），預設 1 秒 |

threshold 計算方式：
```csharp
var ticks = (DateTime.Now.Ticks - 621355968000000000) / 10000 - interval * 1000;
var time = new DateTime(ticks * 10000 + 621355968000000000).ToString("yyyyMMddHHmmss");
// 621355968000000000 = 1970-01-01 的 Ticks
```

---

## 備註

- 原文件描述此表為「在線狀態」用途是不正確的。實際上此表用於**登入驗證次數控制**，不記錄在線狀態。
- TIME 使用字串格式 `varchar(14)` 而非 datetime，便於直接字串比較。
- 登入成功後會清除所有記錄（DELETE WHERE USERID），因此表中只保留「尚未成功登入」的嘗試記錄。
- 若 config 中未設定 `pwdDate.ValidCount`，則不會操作此表（跳過所有檢查/寫入/清除）。
- 使用 `DatabaseType.Split` 存取，表示此表存放在分割資料庫中（非系統資料庫）。
