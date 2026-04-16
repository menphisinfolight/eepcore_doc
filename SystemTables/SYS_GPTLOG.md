# SYS_GPTLOG

## 用途

**GPT AI 使用日誌表**（GPT/AI Usage Log）。

SYS_GPTLOG 設計用於記錄系統中 GPT/AI 功能的使用日誌，包含使用者、提示詞、使用量、類型等，用於 AI 功能的用量統計與成本追蹤。

> ⚠️ **僅存在於 Oracle 建表腳本**：此表定義僅見於 `oracle/systemTables.sql`，SQL Server / MySQL / DB2 / Informix 的建表腳本均未包含。C# 程式碼和前端 JavaScript 也無任何引用。推測為後期新增的預留結構，尚未在程式碼中實作。

### 程式碼現況

- `EEPGlobal.Core/Provider/ChatProvider.cs` 實作了 AI 聊天功能（支援 Azure OpenAI、Claude、OpenAI），但**未引用 SYS_GPTLOG**，目前 AI 功能的使用日誌未被記錄到此表。
- 無對應的 infocommand 定義。
- 無對應的 Provider 操作方法。

---

## 欄位結構（Oracle）

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **SEQID** | `integer` PK | NOT NULL | 自動遞增識別碼 |
| **USERID** | `varchar2(50)` | NOT NULL | 使用者帳號 |
| **PROMPT** | `clob` | NOT NULL | 提示詞/輸入內容 |
| **USED** | `varchar2(50)` | NULL | 使用量（如 Token 數） |
| **TYPE** | `varchar2(20)` | NULL | AI 功能類型 |
| **DATETIME** | `date` | NOT NULL | 使用時間 |

### 對應其他資料庫的推測型別

| 欄位 | Oracle | SQL Server（推測） | MySQL（推測） |
|------|--------|-------------------|--------------|
| SEQID | `integer` + identity | `int IDENTITY(1,1)` | `int AUTO_INCREMENT` |
| PROMPT | `clob` | `nvarchar(max)` | `text` |
| USED | `varchar2(50)` | `varchar(50)` | `varchar(50)` |
| TYPE | `varchar2(20)` | `varchar(20)` | `varchar(20)` |
| DATETIME | `date` | `datetime` | `datetime` |

---

## 主鍵

```
PRIMARY KEY (SEQID)
```

Oracle 建表腳本中有 `createTablesIdentity('SYS_GPTLOG', 'SEQID')` 用於建立序列自動遞增。

---

## 備註

- 此表是目前 EEP Core 系統資料表中唯一僅在 Oracle 腳本中定義的表。
- ChatProvider 支援 Azure OpenAI（gpt-4o）、Claude（claude-3-5-sonnet）、OpenAI、OpenAI-compatible 等模型，未來若需記錄 AI 使用日誌，預期會寫入此表。
- PROMPT 欄位使用 `clob` 可儲存大量文字，適合記錄完整的提示詞內容。
- USED 欄位為 varchar(50)，推測儲存 Token 使用量或費用等數值的字串表示。
