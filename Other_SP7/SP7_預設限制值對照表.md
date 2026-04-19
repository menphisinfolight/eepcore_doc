# SP7 預設限制值對照表

SP7 有一批預設的上限值與 Timeout——筆數、大小、逾時等——散落在不同原始碼、config 與元件裡。這份文件把這類常用預設值集中列表，方便客戶或同仁需要調整時能快速定位原始碼與調整方式。

> 之後如需補充新的預設值，就在這裡加一行；不要把同一段寫進每個元件 doc。

## 對照表

| # | 項目 | 預設值 | 調整方式 | 位置 |
|---|------|--------|---------|------|
| 1 | Excel 匯出筆數 | **1000 筆** | 呼叫 `ExportExcel` 時在 `ExportExcelOptions.PageSize` 傳入較大值（> 0 才生效）；或改底層預設 | `EEPServerTools.Core/Utility/ParserHelper.cs` L793 |
| 2 | InfoCommand Command Timeout | **30 秒** | 在 InfoCommand JSON 設 `CommandTimeout`；或程式碼直接 `helper.CommandTimeout = 180;` | `EEPServerTools.Core/Components/InfoCommand.cs` L71；套用於 `DataModule.cs` L625/637/656/716 |
| 3 | DatabaseHelper Command Timeout | **30 秒** | 屬性直接指派 | `EEPBase.Core/Utility/DatabaseHelper.cs` L90；`EEP.Flow/Sqls/SqlHelper.cs` L44 |
| 4 | IIS 請求內容最大 | **1 GB**（1073741824 bytes） | 改 `web.config` `<requestLimits maxAllowedContentLength="..." />` | `EEPWebClient.Core/web.config` L14 |
| 5 | LDAP search page size | **1000** | 改程式 | `EEPBase.Core/Utility/LDAPHelper.cs` L43；`EEPAdapater/LDAP.cs` L62 |
| 6 | ChatProvider（LLM 呼叫）HTTP Timeout | **10 分鐘** | 改程式 | `EEPGlobal.Core/Provider/ChatProvider.cs` L278 |
| 7 | FileProvider 上傳 body 限制 | **無限**（`long.MaxValue`） | 已用 `[RequestFormLimits]` 放寬，不受 ASP.NET 預設 128MB 限制 | `EEPGlobal.Core/Provider/FileProvider.cs` L477 |

---

## 細節說明

### 1. Excel 匯出筆數（1000）

**常見情境**：客戶按「匯出 Excel」後拿到的檔案筆數固定 ≤ 1000，但查詢清單明明更多。

**實作細節**：`ParserHelper.ExportExcel` L787-793 的邏輯：

```csharp
if (options.PageSize > 0)
{
    queryOptions.PageSize = options.PageSize;   // 呼叫端有傳，就聽它的
}
else
{
    queryOptions.PageSize = 1000;               // fallback = 1000
    ...
}
```

→ **如果呼叫端沒指定 PageSize**，就會硬限 1000。這條限制是**可以用參數覆寫**的，不一定要改底層。

**建議做法**（依優先序）：

1. 呼叫 `ExportExcel(id, new ExportExcelOptions { PageSize = 10000, ... })` — 最乾淨
2. 若該功能無法改呼叫端，才動底層 `ParserHelper.cs` L793；但要注意 **IIS 部署需重新 `dotnet publish`** 才會生效（這條在《IIS 發佈》文件已有通則，此處不重複）

### 2–3. Command Timeout（30 秒）

**常見情境**：複雜 SQL 或大批資料查詢拋 `timeout expired` 例外。

**套用順序**：

```
InfoCommand JSON 的 CommandTimeout
    → DataModule 傳給 DbHelper
    → DbHelper.CommandTimeout 套到每次 cmd 執行
```

因此**優先在 InfoCommand JSON 設定**（如 `"CommandTimeout": 180`），不需改 C#。若是直接 `new DatabaseHelper()` 手動建 helper，則 `helper.CommandTimeout = 180;`。

### 4. IIS 請求內容最大（1 GB）

**常見情境**：超大檔上傳時，IIS 回 **413 Payload Too Large**，此時連 ASP.NET 都還沒接到請求。

**注意**：這是 **IIS 層**的限制，跟 ASP.NET 的 `MaxRequestBodySize` 是兩層（Kestrel 裡那條在 `Startup.cs` L64 目前註解掉）。IIS 託管時以 `web.config` 為準。

### 5. LDAP page size（1000）

**常見情境**：使用 AD 同步時，成員數超過 1000 的群組抓不全。

LDAP server 端本身就有 MaxPageSize（Windows AD 預設也是 1000），client 端若設更大也會被 server 截。通常要搭配 paged cookie 才能真正抓完，EEP 現階段用固定 1000。

### 6. ChatProvider Timeout（10 分）

**常見情境**：呼叫 GPT/Claude 長回應或 streaming 一開始就被中斷。10 分鐘通常夠用，除非跑 long-thinking 的 model。

### 7. FileProvider 上傳（無限）

**說明**：SP7 已主動在該 endpoint 加 `[RequestFormLimits(MultipartBodyLengthLimit = long.MaxValue)]`，放寬 ASP.NET 預設的 128MB 表單限制。**實際上限仍受 IIS 層的 1GB 規範**（見第 4 條）。

---

## 如何維護這份表

- **新增條目**：補一行到對照表，再視情況加一段細節說明。欄位：項目 / 預設值 / 調整方式 / 位置（含行號）。
- **行號漂移**：SP7 升版後行號會動，維護時以 grep 重找（關鍵字：`PageSize`、`CommandTimeout`、`maxAllowedContentLength`、`MultipartBodyLengthLimit` 等）。
- **不要複製到別處**：各元件 doc 只要 cross-reference 回這份表（例如 `ParserHelper.md` 備註末端加一句「完整限制清單見《SP7 預設限制值對照表》」），不要把條目複製貼到每支元件 doc。

---

## 候選（待補 / 待確認）

以下是可能存在但本版尚未確認的預設值，之後確認後再補：

- RWD 表單 `Datagrid` 預設 PageSize
- 檔案附件下載緩衝大小
- 流程簽核並發上限
- Log 保留天數
- Session / Cookie 有效期
- `.security` 檔規則數量上限
