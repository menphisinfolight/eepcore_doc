# InfoPush

> `EEPServerTools.Core/Components/InfoPush.cs` — 63 行
> 繼承：`ServerComponent` → `Component`

> 🏷️ **選購模組功能**：InfoPush 屬於 **MAUI / APP 模組** 的配套元件（討論區 [#471492](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=471492)：「[訊息] 主要是提供予 [APP 模組] 接收推播訊息作使用」）。未購買 APP 模組的部署可以放 InfoPush 元件，但推播不會到達手機。
>
> ℹ️ **公版原始碼中的 `PushHelper` 呼叫為註解狀態**（`InfoPush.cs:59`），推測選購 MAUI 模組時，安裝包會啟用該呼叫或以完整版取代此檔案。公版環境下 `Send()` 會固定 `return ""`；若需在非 MAUI 環境下推播，改走 `PushHelper.sendPushNotification`（見 [#475009](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=475009)）。

## 用途

**推播通知元件**（Info Push）。

設計時間在 Server 模組中宣告推播對象（UserList）、標題、內文、徽章數、連結；執行時由 `PushHelper` 呼叫 Firebase Cloud Messaging（FCM）發送到使用者的 MAUI / Android / iOS App。

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

## 實際使用範例（討論區彙整）

> 原始碼倉 `/mnt/d/EEPCore_SP7_TEST` 沒有 InfoPush 的使用範例；以下來自討論區 2023 年的實戰。**這些範例全部是舊 EEP.NET Framework 版 JavaScript ServerMethod**，寫法與 SP7 C# ServerMethod 不同 ——
>
> - 舊版 JSON key：`toUser` / `title` / `message` / `link`（camelCase）
> - SP7 C# 版 JSON key：`UserList` / `Title` / `Body` / `Link`（PascalCase）— 見 `InfoPush.cs:33-56`
>
> 移植到 SP7 時欄位名要改。公版 Send() 為 stub（見頂端說明）；若未購買 MAUI 模組但仍需推播，改呼叫 `PushHelper.sendPushNotification`。

### 範例 1：推播給單一使用者（來源 [#468471](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468471)，舊 JS 版）

```javascript
// 舊 EEP.NET（Framework）JavaScript ServerMethod —— 直接抄會爆，僅供邏輯參考
exports.AppNotify = function(param, callback) {
    var dm = this;
    dm.getComponent('InfoPush1', 'infopush', function(err, push) {
        if (err) callback(err);
        else {
            var content = {
                toUser: 'nadine',             // ← SP7 改 UserList
                title: '去化單號：xxx 通知提醒',  // ← SP7 改 Title
                message: 'xxx 已去化...',       // ← SP7 改 Body
                link: 'bootstrap/物件去化申請單' // ← SP7 改 Link
            };
            push.Send(content, function(err) {
                if (err) callback(err);
                else callback(null, '推播成功');
            });
        }
    });
};
```

### 範例 2：撈 USERS 表逐筆推播給群組（來源 [#468486](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468486)，舊 JS 版）

```javascript
// 舊版做法：想推給「群組」時沒有直接語法，只能 SELECT USERID 再 loop
exports.APPNotifytest1 = function(param, callback) {
    var dm = this;
    var async = dm.getModule('async');
    dm.getComponent('InfoPush1', 'infopush', function(err, push) {
        var sql = "SELECT USERID FROM USERS WHERE AUTOLOGIN <> 'X'";
        dm.queryRaw(dm.clientInfo, dm.clientInfo.database, sql, {}, function(err, rows) {
            async.eachSeries(rows, function(row, callback2) {
                var content = { toUser: row.USERID, title: '...', message: '...', link: '...' };
                push.Send(content, callback2);
            }, callback(null, '推播成功'));
        });
    });
};
```

### 範例 3：推播到流程活動裡（討論串 [#468458](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468458) / [#468486](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468486)）

- 流程活動設定的 `onExecute` 事件裡可以呼叫推播方法
- **已知坑**：如果流程送出後即執行程序結案，可能沒有 call 到推播；討論區建議「可以把行政人員關卡加回來」避免送出即結案的時機問題

### SP7 .NET Core 推薦路徑（原始碼 `DataModule.cs:881-887`）

```csharp
// ServerMethod 裡直接走 DataModule.CallMethod，Method 名固定為 sendPushNotification，
// 這是框架內建、會真的跑 PushHelper 的路徑（見 DataModule.cs CallMethod 對該 Method 的特判）
CallMethod("sendPushNotification", new JObject {
    { "UserList", "user01,user02" },
    { "Title",    "新簽核通知" },
    { "Body",     "您有一筆待簽核文件" },
    { "Count",    1 },
    { "Sender",   "system" },
    { "Link",     "/approval/detail?id=123" }
});
```

`DataModule.CallMethod` 對 `sendPushNotification` 有特判：**會跳過 `CheckMethod` 權限檢查**（line 883-886），方便框架內部與流程活動直接呼叫。

## 部署 / 設定

| 項目 | 說明 | 來源 |
|------|------|------|
| **FCM 服務帳號** | Firebase `service-account.json` 要放在 **EEPWebClient 的 `bin/` 目錄下**，否則 `PushHelper` 找不到憑證會報錯 | [#475113](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=475113) |
| **Oracle 11g 4000 byte 限制** | `PushHelper.sendPushNotification` 舊版直接拼 SQL insert `SYS_MESSENGER`，遇到大字串在 Oracle 11g 會 `ORA-01704`；新版改用參數化修正 | [#475009](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=475009) |
| **Android 收不到** | iOS 收得到、Android 收不到通常是 Server 端 FCM 檔案版本不對，需更新 `PushHelper` 相關檔 | [#477364](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=477364) |
| **手機顯示與網頁訊息不同** | 自訂推播訊息需由後端 ServerMethod 發送，不要用流程內預設的推播文字 | [#468431](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468431) |

## 備註

- 繼承自 `ServerComponent`，可存取 `ClientInfo` 等伺服器端上下文資訊。
- `Count` 使用 `[NumberboxEditor(1, true, 1)]`，第一個參數為預設值 1，第二個為是否必填（true），第三個為遞增步長 1。
- 主頁 [訊息] 區塊（`EEPWebClient.Core/Views/Main/Sysmessage.cshtml`）也是 APP 模組的推播收件匣；沒買 APP 模組時也可以直接改此 view 做一般系統通知（[#471492](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=471492) 官方建議）。
