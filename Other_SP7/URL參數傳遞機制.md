# URL 參數傳遞機制（SP7）

EEP SP7 網址參數的 6 種來源 + 3 套安全機制（MD5 簽章 / sessionStorage token / Rijndael 密文）完整對照。

---

## 一、6 種參數來源

| # | 來源 | 組網址的地方 | 是否自動加驗證 |
|---|------|--------------|-----------------|
| 1 | Menu 點擊（主選單 / 子頁籤） | 後台依 Menu 設定組 | 自動帶 `_lang` / `_site` / `im` |
| 2 | Flow 起單 / 流程跳頁 | 流程引擎組 `?p=..&flowid=..&stepid=..&formid=..&userid=..` | 依 Security control 設定 |
| 3 | DataGrid ToolItem / Drilldown | JS `openForm` / `openTab` | 改走 **sessionStorage token**（見下） |
| 4 | Server `Response.Redirect` | `MainController.cs:28` | 直接保留既有 QueryString |
| 5 | 前端 JS（`location.href` / `location.replace`） | 自定 JS | 開發者自控 |
| 6 | 書籤 / 手打 | 使用者 | 無 |

---

## 二、3 套安全機制總覽

> **不能混淆**：這三套的保護目標、能否還原參數值、誰解碼，完全不同。

| 特性 | MD5 簽章 `im` / `pm` / `m` | sessionStorage token `?p=xxx` | Rijndael 免登入 `?p=<base64密文>` |
|------|---------|---------|---------|
| 型態 | 不可逆 hash（簽章） | 隨機 8-hex key → 本機查表 | 可逆 AES-ECB 密文 |
| 主要用途 | 驗證 Menu / 頁面 / Activity 權限 | DataGrid Drill、openForm 跨頁傳複雜物件 | 免登入公開連結 |
| Payload 存哪 | 不帶 payload（只簽 key） | 瀏覽器 sessionStorage | URL 本身（base64 編碼密文） |
| 能還原原值 | ❌ | ✅（只有 client 能讀） | ✅（server 端解） |
| Server 看得到原值 | 直接看 URL 明文 | ❌ 看不到（純 client） | 解密後看到 JSON |
| 關鍵 API | `MD5(x)` | `$.getEncryptParameters(key, remove)` | `ConfigHelper.DecryptDES()` |
| 依賴配置 | 無 | 無 | `config.json` 的 `logonKey` |
| 典型看到的網址 | `?menuid=M01&im=a1b2c3...` | `?drill=a3f7b2c1` | `?p=MIGfMA0GCSqGSIb3...` |

---

## 三、詳解：MD5 簽章（`im` / `pm` / `m`）

### 三個參數

| 參數 | 意義 | 計算 | 程式位置 |
|------|------|------|----------|
| `im` | Menu 權限驗證 | `MD5(MENUID)` | `EEPRWDTools.Core/RWDPage.cs:74-77` |
| `pm` | 頁面可存取驗證 | `MD5(p)` | `EEPRWDTools.Core/RWDPage.cs:182-185` |
| `m` | Activity 級欄位/控制項權限 | `MD5(p + ":" + aKey)` | `EEPRWDTools.Core/RWDPage.cs:289-299` |

### 特性

- **只是簽章**，不能還原參數值。使用者仍看得到 `?menuid=M01` 的明文
- 防的是「竄改 menuid 嘗試繞過權限」，不是「藏參數」
- 沒設 Security control 的頁面不會強制驗證 `im`/`pm`，URL 可被任意手改

---

## 四、詳解：sessionStorage token（**真正能藏參數的機制**）

> 這是日常開發裡最常用到的一套，但外觀最容易被誤認為「加密」。

### 流程（5 步）

```
使用者點 DataGrid Drill / 某按鈕
    ↓
(a) JS 產生隨機 token：getNewID()  → 8-hex（例 "a3f7b2c1"）
    ↓
(b) sessionStorage[token] = JSON.stringify(原始參數物件)
    ↓
(c) 跳頁：URL 只帶 ?drill=a3f7b2c1（或 ?p=xxx / ?tabParam=xxx）
    ↓
(d) 目標頁 init：$.getEncryptParameters(key, remove=true)
    ↓
(e) JSON.parse → 還原原始物件，render
```

### 關鍵程式

**產 token**：`EEPWebClient.Core/wwwroot/js/infolight/bootstrap.infolight.js:21367-21373`

```javascript
function getNewID() {
    var id = '';
    for (var i = 1; i <= 8; i++) {
        id += Math.floor(window.crypto.getRandomValues(new Uint8Array(1)) / 256 * 16.0).toString(16);
    }
    return id; // 例 "a3f7b2c1"
}
```

**DataGrid Drill 存 payload**：`bootstrap.infolight.js:13381-13389`

```javascript
var drillRow = {
    targetRemoteName: opts.targetRemoteName,
    whereItems: whereItems,
    drillRow: row,
    loadAction: opts.pageOpenForm ? 'viewRow' : null
};
sessionStorage[id] = JSON.stringify(drillRow);
var url = '../bootstrap/' + opts.page + '?drill=' + id;
```

**目標頁還原 API**：`bootstrap.infolight.js:1914-1928`

```javascript
$.getEncryptParameters = function(key, remove) {
    var p = $.getParameter(key || 'p');      // 預設讀 ?p=，可指定其他 key
    if (p) {
        var parameter = sessionStorage[p];
        if (parameter) {
            if (remove) sessionStorage.removeItem(p);
            var obj = JSON.parse(parameter);
            obj.__p = p;                     // 保留 token 供後續使用
            return obj;
        }
    }
    return null;
};

// 取單一欄位
$.getEncryptParameter = function(name, key) {
    var parameters = $.getEncryptParameters(key);
    return parameters ? parameters[name] : '';
};
```

### 目前 EEP 使用的 key 名稱

| URL 參數 | 用途 | 呼叫點 |
|----------|------|--------|
| `?drill=xxx` | DataGrid Drilldown | `bootstrap.infolight.js:13389` |
| `?p=xxx` | 預設（Flow / openForm 跨頁） | `bootstrap.infolight.js:3568, 6427, 6856, 6960, 20878` |
| `?tabParam=xxx` | 子頁籤傳值 | `bootstrap.infolight.js:20868` |

也有處理 `chatID`（字首 `##`）的分支（`bootstrap.infolight.js:2989, 5152, 7580`），對應的 payload 一樣進 sessionStorage。

### 驗證方式

1. F12 → Application → **Session Storage**
2. 點任何 DataGrid Drill → 會看到一組新 key（8-hex），value 是 JSON
3. 網址列只剩 `?drill=xxx` 或 `?p=xxx`，真實參數都在瀏覽器本機

### 限制

| 項目 | 說明 |
|------|------|
| 生命週期 | 分頁關閉即消失（sessionStorage 定義如此） |
| 同源共享 | 同 origin 的任何 JS 都讀得到 |
| 無法書籤 / 無法複製 URL 給別人 | 另一個瀏覽器拿到 `?drill=xxx` 是空的 |
| Server 看不到 | 後端若要驗條件需靠後續 AJAX 送 payload |
| 沒 TTL 控制 | 參數殘留直到分頁關、手動 clear 或 `remove=true` 用完刪 |

---

## 五、詳解：Rijndael 密文（免登入 URL）

### 使用場景

- 公開連結（不走登入頁）
- 跨系統 SSO
- 免登入網址短網址

### 流程

```
(1) 後台產生密文（EncryptDES）
    明文 JSON: {"database":"TEST", "user":"u001", "solution":"S1", "timeout":<unix_ms>, "redirect":"/xxx"}
    ↓ Rijndael AES-ECB + PKCS7 + key = config.json.logonKey
    ↓ base64
    URL: ?p=MIGfMA0GCSqGSIb3...
       ↓
(2) 瀏覽器打開 → MainController / LogonWithKey
    ↓ 讀 HttpContext.Request.Query["p"]
    ↓ ConfigHelper.DecryptDES(data, logonKey)
    ↓ 驗證 timeout 時戳
    ↓ 建立 Session
    進頁面
```

### 關鍵程式

**`EEPBase.Core/Utility/ConfigHelper.cs:160-233`**

```csharp
// L160 配置
var logonKey = config["logonKey"]?.ToString();

// L175 加密入口
return EncryptDES(data, logonKey);

// L189 解密入口
var data = DecryptDES(key, logonKey);

// L204-218 加密實作
public static string EncryptDES(string toEncrypt, string key) {
    byte[] keyArray = UTF8Encoding.UTF8.GetBytes(key);
    byte[] toEncryptArray = UTF8Encoding.UTF8.GetBytes(toEncrypt);
    RijndaelManaged rDel = new RijndaelManaged();
    rDel.Key = keyArray;
    rDel.Mode = CipherMode.ECB;
    rDel.Padding = PaddingMode.PKCS7;
    ...
}

// L220-233 解密實作對稱
public static string DecryptDES(string toDecrypt, string key) { ... }
```

### config.json 範例

```json
{
  "logonKey": "your-16-or-32-byte-key-here",
  "nonlogons": [
    { "url": "/index.html", "database": "SYSTEM", "solution": "Admin" }
  ]
}
```

### 特性

- **Server 端對稱加解密** → 能完整藏參數值
- 內建 `timeout` 時戳 → 過期 URL 自動失效
- 方法名叫 `DES` 但實際是 **Rijndael (AES)**（歷史命名）
- **key 洩漏 = 全部舊連結可解**，logonKey 要保密、不同環境不要共用

---

## 六、Server / Client 取參數的標準寫法

### Server 端

**RWDPage / ServerMethod 裡**：
```csharp
// RWDPage 實例有 Query 字典
var value = Query["keyName"];    // RWDPage.cs:46
```

**一般 Controller**：
```csharp
HttpContext.Request.Query["keyName"];
HttpContext.Request.Query.ToJObject();   // MainController.cs:67, 122
```

### Client 端（JS）

**讀 URL 參數**：
```javascript
$.getParameter('keyName');       // bootstrap.infolight.js:1936
```

**讀 sessionStorage 還原後的參數**：
```javascript
// 整包還原
var params = $.getEncryptParameters();            // 預設讀 ?p=
var params = $.getEncryptParameters('drill');     // 讀 ?drill=
var params = $.getEncryptParameters('drill', true); // 讀完即刪

// 單一欄位
var v = $.getEncryptParameter('flowid');
```

> **沒有內建 `EEP.getQueryString()` 全域函式**，就是用 `$.getParameter(...)`。

---

## 七、Menu / Flow 附加的自動參數

### Menu 自動帶
| 參數 | 意義 |
|------|------|
| `_lang` | 語系 |
| `_site` | Site ID |
| `im` | MD5(menuid) 驗 Menu 權限 |

Menu 設定面板有「參數」欄位可手動補 QueryString（discuss #480733）。

### Flow 起單典型網址

```
/form?p=<FormName>&flowid=<ID>&stepid=<ID>&formid=<KeyValue>&userid=<User>
```

- 流程參數存 `FLOWTODO.FORM_PRESENTATION_CT`（可帶 HTML）
- 流程設定可自動帶 KEY 欄位（#478692）

---

## 八、常見陷阱

| # | 症狀 | 成因 | 討論 |
|---|------|------|------|
| 1 | 加簽後取不到流程參數 | SP5 bug | #477429 |
| 2 | FLOWTODO 過長被切 | 欄位長度限制 | #482553 |
| 3 | 流程參數重複顯示 | 某版 bug | #481545 |
| 4 | SSO URL 暴露 DB name、user | 沒走 Rijndael 或 key 內容帶敏感資料 | #478723 |
| 5 | iframe 嵌 SSO URL 失敗 | 跨域 / X-Frame-Options | #482584 |
| 6 | 免登入短網址解不出來 | logonKey 不一致 / URL encode 壞 | #482196 |
| 7 | 想把參數藏起來（URL 要變乾淨） | 解法就是 sessionStorage token | #480801 |
| 8 | openForm 傳參語法 | 參數寫法 | #482477 |
| 9 | Drilldown 傳參設定 | 網址欄位設定 | #479352 |
| 10 | 以網址預帶表單值 | 明文參數設計 | #480819 |
| 11 | Refval 來源是 SP 時傳參 | SP 參數綁定 | #480882 |
| 12 | 程式自動起單 sample | 直接 call API | #482510 |

---

## 九、⚠️ 安全提醒

**「看起來有加密」不代表安全**：

| 機制 | 能防 | 不能防 |
|------|------|--------|
| MD5 簽章（`im`/`pm`/`m`） | 使用者竄改 URL | 原始 ID / key 明文外洩 |
| sessionStorage token | URL 可見性（貼給別人沒用） | 同源 JS 全可讀；XSS 一洩全洩 |
| Rijndael 免登入 | 外部看 URL / 竄改 | `logonKey` 外洩（等於全部連結可解） |

**敏感值不應該放 URL 或 sessionStorage**（密碼、完整 session token、個資）。真的敏感就：
- 走 Server Session（`HttpContext.Session`）
- 走 POST body + HTTPS
- 不要依賴前端藏值

---

## 十、相關討論

| ID | 主題 | 發佈 |
|----|------|------|
| #482477 | openForm 參數傳遞 | 2026-03-19 |
| #482584 | iframe 加入 RWD 的 SSO URL 錯誤 | 2026-04-16 |
| #482553 | 資料塞入 FLOWTODO 時過長 | 2026-04-09 |
| #482510 | 程式自動起單 Sample Code | 2026-03-26 |
| #482196 | 免登入 URL 的短網址解析不出實際 URL | 2026-01-13 |
| #481545 | 流程參數問題 | 2026-02-12 |
| #480882 | Refval 來源是 SP 時如何傳參數 | 2025-07-03 |
| #480819 | 以網址帶入表單預帶值 | 2025-06-24 |
| #480801 | 帶參數的網址儲存後跳乾淨網址 | 2025-06-22 |
| #480733 | 如何透過選單傳遞參數 | 2025-06-11 |
| #479352 | Drilldown 如何設定網址傳入的參數 | 2025-04-21 |
| #478723 | SSO 的 URL 參數 | 2025-01-13 |
| #478692 | 流程參數主動包含資料表 KEY 值欄位 | 2025-01-28 |
| #477429 | SP5 流程參數無法取得 | 2024-12-13 |
| #474438 | 免登入網址直接填單不經 dgMaster | 2024-07-02 |

---

## TL;DR

1. EEP 的「加密網址」其實有 3 套，**只有 sessionStorage token 與 Rijndael 密文**能真正藏值；`im`/`pm`/`m` 只是簽章。
2. DataGrid Drill / `openForm` 跨頁傳物件 → 預設就是 **token + sessionStorage**，網址只看到 `?drill=a3f7b2c1`。
3. 取值用 `$.getEncryptParameters(key, remove)`；取單欄位 `$.getEncryptParameter(name, key)`。
4. **真的敏感的值**：不要放 URL，不要放 sessionStorage，走 Server Session。
