# InfoMail

> `EEPServerTools.Core/Components/InfoMail.cs` — 52 行
> 繼承：`Component`

## 用途

**郵件發送元件**（Info Mail）。

InfoMail 用於在伺服器端發送電子郵件。設計師設定收件人、主旨、內文、副本及附件等屬性，呼叫 `Send()` 時透過 `EmailHelper` 發送郵件。郵件伺服器設定取自系統組態的 `email` 區段，若未設定則回傳錯誤訊息。

## JSON 設定範例

```json
{
  "type": "infomail",
  "id": "mail1",
  "subject": "通知信件",
  "body": "這是信件內容",
  "to": "user@example.com",
  "cc": "manager@example.com",
  "fromName": "系統通知"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **Subject** | string | 文字 | 郵件主旨 |
| **Body** | string | 文字 | 郵件內文 |
| **To** | string | 文字 | 收件人（Email 地址） |
| **CC** | string | 文字 | 副本收件人 |
| **BCC** | string | 文字 | 密件副本收件人 |
| **FromName** | string | 文字 | 寄件人顯示名稱 |
| **Attachments** | List\<Attachment\> | 集合編輯器 `[CollectionEditor]` | 附件清單（路徑 `filepath` + 檔名 `fileName`） |

## 內部類別

### Attachment

| 屬性 | 類型 | 說明 |
|------|------|------|
| Filepath | string | 附件檔案路徑 |
| FileName | string | 附件顯示檔名（空白時自動取路徑檔名） |

## 主要方法

| 方法 | 回傳 | 說明 |
|------|------|------|
| `Send()` | object | 讀取系統組態 `email` 區段，透過 `EmailHelper` 發送郵件；未設定時回傳 `"Email config not set"` |

## 備註

- 郵件伺服器設定來自 `ConfigHelper.GetConfig()["email"]`，需在系統組態中預先配置。
- `Attachments` 使用 `[CollectionEditor("components", "infoMail", "attachment", "filepath")]`，在設計介面中以集合編輯器呈現，路徑為 `components > infoMail > attachment > filepath`。
- `FromName` 屬性僅設定寄件人顯示名稱，實際寄件地址由系統組態決定。
- 繼承自 `Component`（非 `ServerComponent`），屬於較底層的基礎元件。
