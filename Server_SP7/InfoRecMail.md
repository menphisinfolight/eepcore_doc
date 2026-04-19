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

## POP3 vs IMAP 行為差異（**重要**）

| 面向 | POP3 | IMAP |
|------|------|------|
| 連接埠 | 995（SSL，寫死） | 993（SSL，寫死） |
| 函式庫 | LumiSoft.Net | S22.Imap |
| **取回範圍** | **全部郵件**（含已讀） | **只取未讀**（`SearchCondition.Unseen()`）|
| 伺服器標記 | 不影響伺服器狀態（除非 protocol 設定刪信）| **讀取後自動標記為已讀** |
| 重複讀取 | 每次都拿到同樣郵件 → 易 DoS 自己 | 每封只讀一次，再讀會 `Unseen()` 為空 |
| 伺服器識別 | Host 要以 `pop3` 開頭（大小寫不分）| Host 要以 `imap` 開頭 |
| 其他 Host 開頭 | `throw NotSupportedException` | — |

### 實務選擇

- **做「處理完就算」的任務**（批次讀入工單 / 指令信）→ **IMAP**（自動標記已讀，下次不會再拿）
- **做「歸檔 / 分析」需保留完整歷史** → **POP3**（但要自己做「已處理」記錄避免重複）
- **Gmail / Office 365 現代要求 OAuth 2.0** → 兩種都不相容（需自行擴充，見下方「限制與風險」）

## C# 使用範例

> 遵守 [EEP Server C# 不寫 using/namespace/class](../其他(SP7)/EEP_Server模組建置機制.md) 規則。
> `InfoRecMail` 的 namespace `EEPServerTools.Core.Components` 已自動 using，可直接使用。

### 1. 基本呼叫：讀取未讀信件並回傳

```csharp
public object ReadUnreadMails(dynamic row)
{
    var recMail = this.GetComponent<InfoRecMail>("recMail1");

    // 動態覆寫連線設定（若設計時已填，param 可以不傳）
    var param = new JObject {
        { "host",        "imap.gmail.com" },
        { "mailAccount", "notify@company.com" },
        { "password",    row["mailPassword"].ToString() }   // 從 row 或系統設定取
    };

    var mails = recMail.Receive(param);   // 回傳 JArray
    return mails;   // 前端可直接 render
}
```

### 2. 讀信並批次寫入資料表

**實戰情境**：客戶寄 Email 到指定信箱，系統自動讀取並建立工單。

```csharp
public object ReceiveAndCreateTickets(dynamic row)
{
    var recMail = this.GetComponent<InfoRecMail>("recMail1");
    var param = new JObject {
        { "host",        "imap.mail.company.com" },
        { "mailAccount", "support@company.com" },
        { "password",    row["pwd"].ToString() }
    };

    var mails = recMail.Receive(param);
    var createdCount = 0;

    foreach (JObject mail in mails)
    {
        var subject  = mail["Subject"]?.ToString() ?? "";
        var from     = mail["From"]?.ToString() ?? "";
        var body     = mail["Body"]?.ToString() ?? "";
        var datetime = mail["Datetime"]?.ToString() ?? "";

        // 避免重複建立（IMAP 保險起見，用 Subject+From+Datetime 當唯一鍵）
        var dup = DbHelper.ExecuteDataTable(
            "SELECT 1 FROM TICKETS WHERE MAIL_FROM = " + DbHelper.MarkValue((object)from)
            + " AND MAIL_DATETIME = " + DbHelper.MarkValue((object)datetime)
        );
        if (dup.Rows.Count > 0) continue;

        // 建立工單
        DbHelper.ExecuteNonQuery(
            "INSERT INTO TICKETS (MAIL_FROM, SUBJECT, BODY, MAIL_DATETIME, STATUS, CREATE_DATE) VALUES ("
            + DbHelper.MarkValue((object)from) + ","
            + DbHelper.MarkValue((object)subject) + ","
            + DbHelper.MarkValue((object)body) + ","
            + DbHelper.MarkValue((object)datetime) + ","
            + DbHelper.MarkValue((object)"NEW") + ","
            + DbHelper.MarkValue((object)DateTime.Now.ToString("yyyy/MM/dd HH:mm:ss")) + ")");
        createdCount++;
    }

    return new { total = mails.Count, created = createdCount };
}
```

### 3. 讀信觸發簽核流程（指令信）

```csharp
public object ProcessApprovalMails()
{
    var recMail = this.GetComponent<InfoRecMail>("recMail1");
    var mails = recMail.Receive(null);   // 用設計時設定

    foreach (JObject mail in mails)
    {
        var subject = mail["Subject"]?.ToString() ?? "";

        // 主旨格式：「APPROVE:FORM-20260419-001」 或 「REJECT:FORM-20260419-001」
        if (!subject.Contains(":")) continue;

        var parts  = subject.Split(':');
        var action = parts[0].Trim().ToUpper();
        var formNo = parts[1].Trim();

        if (action == "APPROVE" || action == "REJECT")
        {
            // 呼叫其他 method 處理簽核
            CallMethod("ApproveForm", new JObject {
                { "formNo", formNo },
                { "action", action },
                { "approver", (mail["From"] ?? "").ToString() }
            });
        }
    }
    return "Done";
}
```

### 4. 搭配排程定期讀信

**設計**：在 SYS_SCHEDULE 建一筆排程，Method 填 `模組名.ReceiveAndCreateTickets`，Type=interval，Setting=10（每 10 分鐘跑一次）。配合 `Schedule.Core.exe` 或 `ScheduleHelper` 啟動（見 [Schedule排程機制](../Other_SP7/Schedule排程機制.md)）。

```csharp
// 排程呼叫的 method（不接受 row 參數，或接受空參數）
public object ScheduledReceive(dynamic param)
{
    var recMail = this.GetComponent<InfoRecMail>("recMail1");
    var result = recMail.Receive(null);

    // 若有信才寫 log
    if (((JArray)result).Count > 0)
    {
        DbHelper.ExecuteNonQuery(
            "INSERT INTO MAIL_FETCH_LOG (FETCH_TIME, COUNT) VALUES ("
            + DbHelper.MarkValue((object)DateTime.Now.ToString("yyyy/MM/dd HH:mm:ss")) + ","
            + ((JArray)result).Count + ")");
    }
    return new { count = ((JArray)result).Count };
}
```

### 5. 錯誤處理

`Receive()` 未攔 SMTP/IMAP 協定錯誤，會原樣往上拋。建議自己包 try-catch：

```csharp
public object SafeReceive(dynamic row)
{
    try
    {
        var recMail = this.GetComponent<InfoRecMail>("recMail1");
        return recMail.Receive(null);
    }
    catch (NotSupportedException)
    {
        // Host 開頭不是 pop3 / imap
        throw new EEPException("收信設定錯誤：Host 必須以 pop3 或 imap 開頭");
    }
    catch (Exception e)
    {
        // 連線、認證、解析錯誤
        DbHelper.ExecuteNonQuery(
            "INSERT INTO MAIL_FETCH_LOG (FETCH_TIME, ERROR) VALUES ("
            + DbHelper.MarkValue((object)DateTime.Now.ToString("yyyy/MM/dd HH:mm:ss")) + ","
            + DbHelper.MarkValue((object)e.Message) + ")");
        throw new EEPException("收信失敗：" + e.Message);
    }
}
```

## 限制與風險（⚠️ 重要）

這個元件設計很薄，有明顯限制。使用前務必理解：

### 1. **不處理附件**

原始碼只取四個欄位：`Subject / From / Body / Datetime`。**附件、收件人、CC/BCC、Reply-To、Message-Id 等全部被忽略**。

**影響**：若情境需要處理附件（例如客戶寄 Excel 進來自動匯入），InfoRecMail **不夠用**。要改用直接呼叫 `LumiSoft.Net` / `S22.Imap` 並自行處理 `MIME` 結構。

### 2. **不支援 OAuth 2.0**

用明文密碼走 SMTP 舊式認證。**Gmail 已停用 Basic Auth（需應用程式密碼 + 低安全性 App）、Office 365 2025 年起逐步停用**。

**對策**：
- Gmail：到帳號 → 安全性 → 建立「應用程式密碼」（2-step 開啟後才可建）
- Office 365：Microsoft Graph API 取信（InfoRecMail 不支援，須自製擴充）

### 3. **Port 寫死 995 / 993**

原始碼硬編碼 SSL port。若郵件伺服器用非標準 port（例如企業自建 IMAP port 143 STARTTLS），**無法設定**。

**對策**：需要改框架 `InfoRecMail.cs` 或自寫 Server Method 直接呼叫 `LumiSoft.Net.POP3.Client`。

### 4. **POP3 不標記已讀會重複處理**

POP3 模式每次取全部郵件，**沒有伺服器端「已讀」概念**（除非設定刪除）。連續跑會**重複讀到同一封**。

**對策**（擇一）：
- **改用 IMAP**（自動已讀）
- **自行記錄處理過的 Subject+From+Datetime**，用 `INSERT ... WHERE NOT EXISTS` 避免重複（見範例 2）
- **讀完後呼叫 POP3_Client.DeleteAll()** 清信箱（危險，信會消失）

### 5. **密碼明文儲存**

`Password` 屬性明文存 JSON。**設計時資料 `design/bootstrap/...json` 會包含密碼**，若 git commit 會洩漏。

**對策**：
- 不要在設計時寫死密碼，從系統變數讀：`param["password"] = $.getVariableValue('mailPassword')`
- 或從 SYS_PARAS（加密）欄位讀

### 6. **錯誤訊息不友善**

原始碼 L109 `throw new NotSupportedException("aa")` — 錯誤訊息就是字串 `"aa"`（明顯是開發期 TODO）。除錯時不易看出原因。

**對策**：自己包 try-catch 改寫錯誤訊息（見範例 5）。

### 7. **Datetime 為字串**

回傳的 `Datetime` 是格式化後的字串（`yyyy-MM-dd HH:mm:ss`），不是 `DateTime` 物件。要排序 / 比對要自己 `DateTime.Parse`。

### 8. **Body 格式不一致**

- POP3：優先 `BodyHtmlText`，沒有才 `BodyText`
- IMAP：直接 `message.Body`（由 S22.Imap 決定）

兩種行為略有差異，混用時要自己做正規化。

## 為什麼討論區幾乎沒有相關提問？

搜全庫只有 **1 篇**（#468252）提到收信相關且還是問寄件者，不是用 InfoRecMail。推測：

- **情境少**：多數企業「寄出」郵件遠多於「讀入」
- **限制多**：不支援 OAuth / 附件 / 收件人 → 不適合現代需求，客戶索性自製
- **沒有前端元件**：設計介面沒對應的即取即看 UI，只能靠 Server Method 呼叫
- **文件偏少**：官方 CHM 也少提，使用者難發現

實務上較多用法：
- **內部通知信箱自動化**（讀指令信）
- **ERP 整合**（供應商寄報價單進 Email，系統自動匯入）
- **測試寄信功能的收端**（QA 用途）

## 備註

- POP3 和 IMAP 的連接埠為固定值（995 和 993），均使用 SSL 加密連線。
- POP3 模式下載全部郵件；IMAP 模式僅取未讀郵件，行為不同。
- POP3 取得的內文優先使用 HTML（`BodyHtmlText`），若無則退回純文字（`BodyText`）。
- 此元件僅負責收信，寄信功能由其他元件（如 InfoMail）處理。
- **密碼以明文存放**在屬性中，無加密機制 — 不要在設計時硬編碼，建議用系統變數或 SYS_PARAS。
- **不處理附件** — 需要附件改用其他方式（直接呼叫 `LumiSoft.Net` 或 `S22.Imap`）。
- **不支援 OAuth 2.0** — Gmail 需應用程式密碼，Office 365 Graph API 須自行擴充。
- 錯誤處理薄弱（L109 `throw NotSupportedException("aa")`），實務上應自己包 try-catch。
