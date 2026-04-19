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
| **Subject** | string | 文字 | 郵件主旨（UTF-8） |
| **Body** | string | 文字（支援 HTML） | 郵件內文，**預設 `IsBodyHtml = true`**，可直接嵌入 `<html>` / `<img>` / `<table>` 等標籤 |
| **To** | string | 文字 | 收件人 Email；**多個請用分號 `;` 分隔**（非逗號） |
| **CC** | string | 文字 | 副本收件人；**多個請用分號 `;` 分隔** |
| **BCC** | string | 文字 | 密件副本收件人；**多個請用分號 `;` 分隔** |
| **FromName** | string | 文字 | 寄件人顯示名稱（實際寄件地址仍由系統組態的 `From` 決定） |
| **Attachments** | List\<Attachment\> | 集合編輯器 `[CollectionEditor]` | 附件清單（路徑 `filepath` + 檔名 `fileName`） |

> **重要**：Body 永遠以 HTML 寄送（原始碼 `EmailHelper.cs:94` `IsBodyHtml = true`），不需要特別設定。純文字也能跑，但若含 `<`、`>` 會被當標籤。

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

## C# 使用範例

### 1. 從 Server Method 呼叫 InfoMail（基本）

**前端（RWD 按鈕）**：

```javascript
function notify_onclick() {
    $.callSyncMethod('訂單模組', 'SendOrderNotify', {
        orderNo: '20260419-001',
        to: 'user@example.com'
    });
}
```

**後端 Server Method**（[#474434](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474434) 官方 pattern）：

```csharp
public object SendOrderNotify(dynamic row)
{
    // 從設計介面的 InfoMail 元件取出
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["to"].ToString();
    mail.Subject = "訂單通知：" + row["orderNo"];
    mail.Body    = "<p>您好，訂單 <strong>" + row["orderNo"] + "</strong> 已建立。</p>";
    return mail.Send();   // 成功回空 List、失敗回錯誤字串
}
```

### 2. 多收件人 / CC / BCC（`;` 分隔）

[#474076](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474076) / 原始碼 `EmailHelper.cs:97`：

```csharp
public object NotifyTeam(dynamic row)
{
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To  = "alice@a.com;bob@b.com;carol@c.com";   // ★ 分號分隔
    mail.CC  = "manager@a.com;director@a.com";
    mail.BCC = "audit@a.com";
    mail.Subject = "週會通知";
    mail.Body    = "<p>請參考附件。</p>";
    return mail.Send();
}
```

**⚠️ 用逗號 `,` 會被當成單一 Email 字串 → SMTP 會拋錯**（[#474085](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474085) 客戶就踩過類似的 Email 格式坑）。

### 3. HTML Body 含變數替換

[#474451](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474451) / [#474076](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474076)：

```csharp
public object SendFromTemplate(dynamic row)
{
    // 從 DB 讀模板（可放在 SYS_PARAS 或自訂 Table）
    var db = this.DbHelper;
    var tpl = db.ExecuteDataTable(
        "SELECT TEMPLATE_BODY FROM MAIL_TEMPLATE WHERE TEMPLATE_ID = " + db.MarkValue((object)row["tplId"])
    ).Rows[0]["TEMPLATE_BODY"].ToString();

    // 變數替換（HTML 模板裡放 {orderNo} / {userName} / {date}）
    var body = tpl
        .Replace("{orderNo}",  row["orderNo"].ToString())
        .Replace("{userName}", row["userName"].ToString())
        .Replace("{date}",     DateTime.Now.ToString("yyyy/MM/dd"));

    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["to"].ToString();
    mail.Subject = "通知：" + row["orderNo"];
    mail.Body    = body;   // 含 HTML 標籤
    return mail.Send();
}
```

**HTML 模板範例**：

```html
<html>
<body style="font-family: Arial, sans-serif;">
  <h2 style="color: #2c3e50;">訂單通知</h2>
  <p>Dear {userName},</p>
  <p>您的訂單 <strong style="color: #e74c3c;">{orderNo}</strong> 已於 {date} 建立。</p>
  <table border="1" cellpadding="8" style="border-collapse: collapse;">
    <thead>
      <tr style="background: #3498db; color: white;">
        <th>品項</th><th>數量</th><th>單價</th>
      </tr>
    </thead>
    <tbody>
      <tr><td>產品 A</td><td>10</td><td>$100</td></tr>
    </tbody>
  </table>
  <p style="color: #888; font-size: 12px;">此為系統自動發送郵件，請勿直接回覆。</p>
</body>
</html>
```

### 4. 動態附件（非設計時 Attachments 屬性）（[#478962](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478962)）

**情境**：附件路徑依資料庫內容動態決定，不能在設計介面寫死。作法是**在程式中 `.Attachments.Add(...)`**：

```csharp
public object SendWithDynamicAttachments(dynamic row)
{
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["to"].ToString();
    mail.Subject = "報表附件：" + row["reportName"];
    mail.Body    = "<p>報表已產生，請見附件。</p>";

    // 查 DB 取出動態附件清單
    var dt = this.DbHelper.ExecuteDataTable(
        "SELECT FILE_PATH, DISPLAY_NAME FROM REPORT_FILES WHERE REPORT_ID = "
        + this.DbHelper.MarkValue((object)row["reportId"])
    );

    foreach (DataRow r in dt.Rows)
    {
        mail.Attachments.Add(new InfoMail.Attachment
        {
            Filepath = r["FILE_PATH"].ToString(),
            FileName = r["DISPLAY_NAME"].ToString()  // 留空會用路徑檔名
        });
    }

    return mail.Send();
}
```

### 5. 夾帶 FileUpload 欄位上傳的檔案（[#480563](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480563)）

FileUpload 欄位存的是 `design/files/{filename}` 這類路徑：

```csharp
public object SendProfileToMember(dynamic row)
{
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["email"].ToString();
    mail.Subject = "員工資料";
    mail.Body    = "<p>您好，以下為您的員工資料。</p>"
                 + "<img src=\"cid:photo\" style=\"max-width:200px;\"/>"
                 + "<p>姓名：" + row["userName"] + "</p>";

    // 從 fileupload 欄位取路徑（通常為 "檔名.jpg" 需組合 design/files/ 前綴）
    var photoPath = "design/files/" + row["PHOTO_FILE"].ToString();
    mail.Attachments.Add(new InfoMail.Attachment { Filepath = photoPath, FileName = "photo.jpg" });

    return mail.Send();
}
```

### 6. 信件內嵌圖片（cid:）

```csharp
public object SendWithInlineImage(dynamic row)
{
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["to"].ToString();
    mail.Subject = "包含 Logo 的通知";
    mail.Body    = @"
        <html><body>
          <img src='cid:logo' style='width:200px;'/>
          <h2>系統通知</h2>
          <p>訂單已成立。</p>
        </body></html>";

    // 附件檔名 = HTML 的 cid
    mail.Attachments.Add(new InfoMail.Attachment
    {
        Filepath = "design/files/company-logo.png",
        FileName = "logo"
    });
    return mail.Send();
}
```

> ⚠️ SP7 升版後有回報**圖片嵌入失效**的案例（[#482273](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482273)），官方有出修正檔（參考 [#482160](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=482160) 16 樓）。若遇到，需更新 Provider 檔並 rebuild。

### 7. 呼叫結果判斷（成功 / 失敗）

```csharp
public object TrySendAndLog(dynamic row)
{
    var mail = this.GetComponent<InfoMail>("mail1");
    mail.To      = row["to"].ToString();
    mail.Subject = "測試";
    mail.Body    = "<p>測試內容</p>";

    var result = mail.Send();

    // 成功：回傳 List<object>（空集合）
    // 失敗：回傳 string（"寄送失敗: ..." / "SMTP錯誤: ..." / "一般錯誤: ..."）
    if (result is string errMsg && !string.IsNullOrEmpty(errMsg))
    {
        // 寫 log 或追加到自訂通知表
        this.DbHelper.ExecuteNonQuery(
            "INSERT INTO MAIL_FAIL_LOG (TO_EMAIL, ERR_MSG, SEND_DATE) VALUES ("
            + this.DbHelper.MarkValue((object)mail.To) + ","
            + this.DbHelper.MarkValue((object)errMsg) + ","
            + this.DbHelper.MarkValue((object)DateTime.Now.ToString("yyyy/MM/dd HH:mm:ss")) + ")");
        throw new EEPException("寄信失敗：" + errMsg);
    }
    return "寄信成功";
}
```

## Mail Server 系統組態

`email` 區段來自 `design/config/system.json`（或類似），由 EEP 的「工具 → 設定 → 系統設定」維護（[#469512](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469512) 官方引導）。

### 傳統 SMTP（SmtpClient）

| 設定 | 說明 |
|------|------|
| `Host` | SMTP 伺服器（例：`smtp.gmail.com`、`smtp.office365.com`） |
| `Port` | 通常 `587`（TLS）或 `465`（SSL） |
| `EnableSsl` | 依伺服器而定 |
| `From` | 寄件地址（實際 `From`） |
| `UserName` | 顯示名稱（實際寄件人顯示） |
| `Password` | SMTP 登入密碼（**Gmail 必須用「應用程式密碼」**，不是帳號密碼—[#481454](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481454)） |

### OAuth 2.0（Office 365 / Microsoft Graph，自 2025-02 起支援）

> 來源：[#479033](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479033)

新增三個欄位：

| 設定 | 說明 |
|------|------|
| `TenantId` | Microsoft Entra（原 Azure AD）租用戶 ID |
| `ClientId` | 註冊的應用程式 ID |
| `ClientSecret` | 應用程式用戶端密碼 |

設定後：
- Office 365（Host 含 `office`）走 **Microsoft Graph API** 發信（`SendMailKit`，原始碼 L155）
- 非 Office 則走 MailKit SMTP with XOAUTH2
- 留空這三個欄位 → 走傳統 `SmtpClient`

## 錯誤處理

### 錯誤日誌位置

發送失敗會記在：

```
EEPWebClient.Core/emailError.log
```

**來源**：[#474085](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474085) Roland 明確指出這個位置。

### 常見錯誤對照

| 錯誤訊息前綴 | 原因 | 對策 |
|------------|------|------|
| `寄送失敗: {email} ...` | 特定收件人失敗（SmtpFailedRecipientException） | 檢查 Email 格式、是否存在 |
| `SMTP錯誤: {StatusCode} ...` | SMTP 伺服器拒絕（SmtpException） | 看 StatusCode：認證失敗、連線失敗、速率限制 |
| `一般錯誤: ...` | 其他未預期錯誤 | 多半是附件讀取失敗、檔案不存在 |
| `Email config not set` | 系統組態未設定 `email` 區段 | 到「工具 → 設定 → 系統設定」設 Mail Server |

### ⚠️ 無拋例外的失敗（重要陷阱）

[#474085](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474085)：寄件地址含單引號（如 `'abc@abc.com'`）導致寄送失敗，**但 `Send()` 不會 throw 例外**，只會回傳字串錯誤訊息。使用者以為成功但實際沒寄出。

**對策**：
- 在呼叫 `Send()` 後檢查回傳型別（見上方範例 7）
- 重要郵件加一個「驗證活動」前置檢查 Email 格式（`MailAddress.TryParse`）
- 固定看 `emailError.log` 監控

## 常見陷阱與限制（討論區彙整）

| 陷阱 | 說明 | 對策 | 來源 |
|------|------|------|------|
| **多收件人用逗號導致失敗** | 原始碼用 `;` split，逗號會被視為單一字串 | 統一用 `;` | 原始碼 L97 |
| **寄件地址含單引號不會報錯** | Send 回傳錯誤字串、不拋例外 | 檢查回傳型別 / 查 emailError.log | [#474085](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474085) |
| **Gmail 密碼用一般密碼失敗** | 要用**應用程式密碼** | 去 Google 帳號 → 安全性 → 建立應用程式密碼 | [#481454](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481454) / [#481580](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481580) |
| **SP7 升版後圖片嵌入失效** | 圖片 `<img src="cid:xxx">` 沒顯示 | 更新 [#482160](https://www.infolight.com/cloud_andyhome_bootstrap/DISCUSSDT?type=010&id=482160) 16 樓的 Provider 檔並 rebuild | [#482273](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482273) |
| **Body 純文字意外渲染為 HTML** | `IsBodyHtml = true` 寫死 | 若要純文字，HTML escape `< > &` 或用 `<pre>` 包 | 原始碼 L94 |
| **附件 .msg 檔案亂碼** | FileUpload 處理 .msg 問題 | 設定 FileUpload 的 `DropEnable = false` | [#481153](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481153) |
| **排程寄信不定時漏信** | SMTP 被 throttle / 連線釋放太快 | 迴圈中加 `Thread.Sleep` 或改 `Task.Delay` | [#474225](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474225) |
| **delaySendMail 排程不觸發** | Schedule.Core.exe 沒啟動（見 EEP_Schedule機制 文件） | 確認 exe 在跑 / 看 SYS_SCHEDULE_LOG | [#471182](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=471182) |
| **Email subject not set** | 流程呈送時 Subject 空值 | 流程活動設定 Email 屬性時 Subject 必填 | [#480427](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480427) |
| **InfoMail 元件只能放固定附件** | 設計介面 Attachments 是靜態集合 | 程式中 `.Attachments.Add(...)` 動態加（範例 4） | [#478962](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478962) |
| **附件路徑用絕對 / 相對路徑** | 相對路徑以 EEPWebClient.Core 根目錄為基準 | 建議用 `design/files/xxx` 或絕對路徑 | 原始碼 L122 |
| **FromName 改不了實際寄件地址** | 只能改顯示名稱，From 由系統組態鎖死 | 要換寄件地址就改系統設定 | 原始碼 L89 |
| **Office 365 用一般 SMTP 認證失敗** | Microsoft 在逐步禁用 Basic Auth | 改用 OAuth 2.0（TenantId/ClientId/ClientSecret） | [#479033](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=479033) |
| **多國語系郵件內容** | Body 寫死一種語言 | 依 `ClientInfo.Locale` 選不同模板 | [#476306](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=476306) |

## 備註

- 郵件伺服器設定來自 `ConfigHelper.GetConfig()["email"]`，需在「工具 → 設定 → 系統設定」預先配置。
- `Attachments` 使用 `[CollectionEditor("components", "infoMail", "attachment", "filepath")]`，在設計介面中以集合編輯器呈現，路徑為 `components > infoMail > attachment > filepath`。
- `FromName` 僅設定寄件人顯示名稱，實際寄件地址由系統組態的 `From` 決定。
- 繼承自 `Component`（非 `ServerComponent`），屬於較底層的基礎元件。
- **Body 永遠 HTML** — 可直接嵌 `<img>` / `<table>` / `<style>`；若要嵌圖請用 `cid:` + 對應附件（範例 6）。
- **收件人 / CC / BCC 皆 `;` 分隔**（不是 `,`，容易踩坑）。
- **成功回傳空 List、失敗回傳錯誤字串**（不拋例外）— 務必檢查回傳型別。
- **自 2025-02 支援 OAuth 2.0**，Office 365 建議改 OAuth。
- **失敗記在 `emailError.log`**（EEPWebClient.Core 根目錄）。
