# InfoRecMail

> `EEPServerTools.Core/Components/InfoRecMail.cs` — 126 行
> 繼承：`ServerComponent` → `Component`

## 用途

**收信元件**（Receive Mail）。

InfoRecMail 用於從郵件伺服器接收電子郵件。支援 POP3（port 995, SSL）和 IMAP（port 993, SSL）兩種協定。呼叫 `Receive()` 方法後回傳 JArray 格式的郵件清單，每封郵件包含主旨（Subject）、寄件人（From）、內文（Body）、日期時間（Datetime）四個欄位。

- **POP3 模式**：使用 `LumiSoft.Net` 函式庫，取回信箱中所有郵件。
- **IMAP 模式**：使用 `S22.Imap` 函式庫，僅取回未讀郵件（`SearchCondition.Unseen()`）。

## JSON 設定範例

```json
{
  "type": "inforecmail",
  "id": "recMail1",
  "host": "pop3.example.com",
  "mailAccount": "user@example.com",
  "password": "xxxxx"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **host** | string | 文字 | — | 郵件伺服器主機位址（以 `pop3` 或 `imap` 開頭決定協定） |
| **mailAccount** | string | 文字 | — | 郵件帳號 |
| **password** | string | 文字 | — | 郵件密碼 |

## 核心方法

### `Receive(JToken param)` → `JArray`

公開方法，供外部呼叫。接收 JToken 參數可動態覆蓋 Host、MailAccount、Password 屬性。回傳 JArray，每個元素為 JObject：

```json
{
  "Subject": "郵件主旨",
  "From": "sender@example.com",
  "Body": "郵件內文（HTML 優先，無 HTML 則取純文字）",
  "Datetime": "2026-04-16 10:30:00"
}
```

### `Receive()` → `List<MailInfo>`（私有）

根據 Host 開頭判斷協定：
- `pop3` → 透過 POP3_Client 連接 port 995（SSL），遍歷所有郵件。
- `imap` → 透過 ImapClient 連接 port 993（SSL），查詢未讀郵件。
- 其他 → 拋出 `NotSupportedException`。

## 內部類別

### `MailInfo`

| 屬性 | 類型 | 說明 |
|------|------|------|
| Subject | string | 郵件主旨 |
| From | string | 寄件人 |
| Body | string | 郵件內文 |
| Datetime | string | 日期時間字串 |

## 備註

- POP3 和 IMAP 的連接埠為固定值（995 和 993），均使用 SSL 加密連線。
- POP3 模式下載全部郵件；IMAP 模式僅取未讀郵件，行為不同。
- POP3 取得的內文優先使用 HTML（`BodyHtmlText`），若無則退回純文字（`BodyText`）。
- 此元件僅負責收信，寄信功能由其他元件（如 InfoMail）處理。
- 密碼以明文存放在屬性中，無加密機制。
