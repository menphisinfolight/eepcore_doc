# SYS_EXTAPPROVE

> ⚠️ **舊版遺留** - 此表在 SQL 腳本中有定義，但 C#、JavaScript、Razor 原始碼均無引用。

## 用途

（無法確認實際用途，原始猜測：外部簽核設定表 External Approve Settings）

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **APPROVEID** | `nvarchar(50)` | 簽核設定識別碼 |
| **GROUPID** | `nvarchar(50)` | 群組/角色 ID（對應 GROUPS.GROUPID） |
| **MINIMUM** | `nvarchar(50)` | 最低金額（觸發此簽核的最低值） |
| **MAXIMUM** | `nvarchar(50)` | 最高金額（觸發此簽核的最高值） |
| **ROLEID** | `nvarchar(50)` | 簽核角色 ID（金額在此範圍內需由此角色簽核） |

---

## 備註

- 此表無明確主鍵定義。
- 典型場景：金額 < 10000 由課長簽核、10000~100000 由經理簽核、> 100000 由副總簽核。
