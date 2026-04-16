# SYS_INFOPAY

> ⚠️ **狀態**：舊版遺留，**未實際使用**
>
> 此表雖在 SQL 腳本中有定義，但 C# 程式碼未引用。JavaScript 中的 InfoPayTypes（金流閘道設定）實際儲存位置待確認。

---

## 用途

**付費資訊表**（Info Payment Record）。

SYS_INFOPAY 儲存系統的付費記錄，包含付款者、期數、金額、內容、退款等資訊。

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **ID** | `nvarchar(20)` PK | 記錄識別碼 |
| **Payer** | `nvarchar(50)` | 付款者 |
| **Term** | `nvarchar(20)` | 期數 |
| **DateTime** | `datetime` | 付款時間 |
| **Amount** | `nvarchar(50)` | 金額 |
| **Content** | `nvarchar(1000)` | 付費內容 |
| **PayRefund** | `nvarchar(10)` | 付款/退款標記 |
| **Remark** | `nvarchar(1000)` | 備註 |

---

## 主鍵

```
PRIMARY KEY (ID)
```
