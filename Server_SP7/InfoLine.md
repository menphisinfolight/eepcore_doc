# InfoLine

> `EEPServerTools.Core/Components/InfoLine.cs` — 45 行
> 繼承：`SelectComp` → `ServerComponent` → `Component`

## 用途

**LINE 訊息發送元件**。

InfoLine 用於從伺服器端發送 LINE 通知訊息。設計師可在模組中設定收件人、主旨與訊息內容，Runtime 時呼叫 `Send()` 方法，透過 `LineHelper` 將訊息發送至指定的 LINE 使用者或群組。發送前會從系統組態 `ConfigHelper.GetConfig()` 讀取 `Lines` 設定，若無設定則不發送。

## JSON 設定範例

```json
{
  "type": "infoline",
  "id": "line1",
  "toUser": "U1234567890abcdef",
  "subject": "通知主旨",
  "message": "這是通知內容",
  "types": "User"
}
```

## 設計介面屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| **id** | string | 元件識別碼（唯一） |
| **toUser** | string | LINE 收件人識別碼 |
| **subject** | string | 訊息主旨 |
| **message** | string | 訊息內容 |
| **types** | LineType | 發送對象類型：`User`（個人）或 `Group`（群組） |

## Core 方法

### Send(JToken param = null)

發送 LINE 訊息的主要方法。流程如下：

```
1. 若傳入 param（JToken），以 param 中的值覆蓋元件屬性：
   - param["toUser"]  → ToUser
   - param["message"] → Message
   - param["subject"] → Subject（注意：原始碼以 message 判斷是否存在，再取 subject 值）
2. 透過 ConfigHelper.GetConfig() 取得系統組態
3. 檢查組態中 Lines 設定是否存在且非空陣列
4. 若有設定，建立 LineHelper 並呼叫 Send() 發送訊息
```

`LineHelper` 建構時傳入 `ClientInfo`（連線資訊）與 `config["Lines"]`（LINE 頻道設定），呼叫時傳入 `ToUser`、`Subject`、`Message` 及 `Types`（轉為字串）。

## LineType 列舉

定義於 `EEPServerTools.Core/Enum.cs`：

| 值 | 說明 |
|------|------|
| `User` | 發送給個別使用者 |
| `Group` | 發送給群組 |

## 備註

- InfoLine 繼承 `SelectComp`，但本身未使用任何查詢相關功能，僅利用基底類別提供的 `ClientInfo` 等基礎屬性。
- `Send()` 方法中存在一個可能的 bug：第 33 行判斷 `param["message"] != null` 後卻取 `param["subject"]` 的值，應為 `param["subject"] != null` 才正確。這代表當 `param` 中有 `subject` 但沒有 `message` 時，`Subject` 不會被覆寫。
- 系統組態中 `Lines` 為空陣列 `"[]"` 時視為未設定，不會發送。
- 實際的 LINE API 串接邏輯封裝在 `LineHelper` 中，InfoLine 僅負責參數組裝與呼叫。
