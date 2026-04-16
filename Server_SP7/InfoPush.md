# InfoPush

> `EEPServerTools.Core/Components/InfoPush.cs` — 63 行
> 繼承：`ServerComponent` → `Component`

## 用途

**推播通知元件**（Info Push）。

InfoPush 用於在伺服器端發送推播通知給指定使用者。設計師設定接收者清單、標題、內文、數量徽章、寄件者及連結等屬性，呼叫 `Send()` 時透過 `PushHelper` 發送推播。

## JSON 設定範例

```json
{
  "type": "infopush",
  "id": "push1",
  "userList": "user01,user02",
  "title": "新簽核通知",
  "body": "您有一筆待簽核文件",
  "count": 1,
  "sender": "system",
  "link": "/approval/detail?id=123"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **UserList** | string | 文字 | 推播接收者清單 |
| **Title** | string | 文字 | 推播標題 |
| **Body** | string | 文字 | 推播內文 |
| **Count** | int | 數字框 `[NumberboxEditor(1, true, 1)]` | 通知數量徽章（預設 1，最小值 1，遞增 1） |
| **Sender** | string | 文字 | 寄件者識別 |
| **Link** | string | 文字 | 點擊通知後的跳轉連結 |

## 主要方法

| 方法 | 回傳 | 說明 |
|------|------|------|
| `Send(JToken param)` | object | 發送推播通知；可透過 `param` JSON 動態覆寫各屬性值 |

### Send() 參數覆寫

`Send()` 接受選用的 `JToken param`，若傳入則會以 JSON 中的欄位值覆寫元件屬性：

```json
{
  "UserList": "user03",
  "Title": "動態標題",
  "Body": "動態內文",
  "Count": 3,
  "Sender": "workflow",
  "Link": "/some/path"
}
```

## 備註

- 目前 `Send()` 中的 `PushHelper` 呼叫已被註解，方法固定回傳空字串，推播功能尚未啟用。
- 繼承自 `ServerComponent`，可存取 `ClientInfo` 等伺服器端上下文資訊。
- `Count` 使用 `[NumberboxEditor(1, true, 1)]`，第一個參數為預設值 1，第二個為是否必填（true），第三個為遞增步長 1。
