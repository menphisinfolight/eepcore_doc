# SYS_FLDEFINITION

> ⚠️ **舊版遺留** - 此表在 SQL 腳本中有定義，但 C#、JavaScript、Razor 原始碼均無引用。

## 用途

（無法確認實際用途，原始猜測：流程定義表 Flow Definition）

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **FLTYPEID** | `nvarchar(50)` PK | 流程類型識別碼（如 `LeaveFlow`、`ExpenseFlow`） |
| **FLTYPENAME** | `nvarchar(200)` | 流程名稱（如 `請假流程`、`費用報銷流程`） |
| **FLDEFINITION** | `ntext` | 流程定義內容（XML 或 JSON 序列化的流程圖、步驟、條件等） |
| **VERSION** | `int` | 版本號 |

---

## 主鍵

```
PRIMARY KEY (FLTYPEID)
```

---

## 備註

- FLDEFINITION 使用 `ntext` 類型儲存完整的流程定義，內容量可能很大。
- VERSION 用於流程版本管理，修改流程時可遞增版本號。
