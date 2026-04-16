# SYS_URL_LIST2

## 用途

**短網址快取表**（Short URL Cache / URL Shortener）。

SYS_URL_LIST2 是 EEP Core 短網址系統的核心快取表，以完整 URL 為鍵儲存對應的短網址。當系統產生分享連結（SSO 登入 URL、免登入 URL）時，自動呼叫第三方短網址服務（Picsee）產生短網址，並快取於此表，避免重複呼叫 API。

### 短網址產生流程

```
使用者點擊「取得 SSO URL」或「取得免登入 URL」
    ↓
MenuProvider / AccountProvider 組合完整 URL
    ↓
GetShortUrl(url)
    ↓
查詢 SYS_URL_LIST2（WHERE URL = 原始 URL）
    ├── 已存在 → 直接回傳 ShortURL（快取命中）
    └── 不存在 → 呼叫 Picsee API 產生短網址
                  ↓
              GetPicsUrl(url)
              POST https://api.pics.ee/v1/links/?access_token=...
                  ↓
              回傳 picseeUrl（如 https://picsee.io/xxxxx）
                  ↓
              寫入 SYS_URL_LIST2（INSERT）
                  ↓
              回傳 ShortURL
```

### 使用場景

| 場景 | 說明 |
|------|------|
| **SSO 分享連結** | 設計工具中「取得 SSO URL」，產生含 SSO Token 的短網址，讓使用者免登入直接開啟頁面 |
| **免登入連結** | 「取得免登入 URL」，產生不需登入即可存取的短網址 |
| **登入頁短網址** | AccountProvider 的 GetLogonUrl，將登入頁 URL 縮短 |
| **QR Code** | 前端 showUrlDialog 將短網址轉為 QR Code 顯示，方便手機掃碼 |

### 前端 UI

設計工具中點擊「SSO URL」或「免登入 URL」後，彈出對話框：
- 左側：URL 文字（可切換短網址/原始 URL）
- 右側：QR Code（由 jquery.qrcode 產生）
- 底部：`☑ 短網址` 切換核取方塊

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPGlobal.Core/Provider/MenuProvider.cs` | 核心：GetShortUrl、GetPicsUrl、GetSSOAddress、GetNonLogonAddress |
| `EEPGlobal.Core/Provider/AccountProvider.cs` | 登入頁：GetLogonUrl → GetShortUrl |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.designer.js` | 前端：getSSOAddress、getNonLogonAddress、showUrlDialog、QR Code |

### 第三方服務

- **Picsee**（https://api.pics.ee）：台灣短網址服務
- API：`POST https://api.pics.ee/v1/links/?access_token=c95f0a29fd852e0d39d07a287a3cdf034b69f9d1`
- 回傳：`data.picseeUrl`（如 `https://picsee.io/xxxxx`）
- externalId：`infolight`

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **URL** | `nvarchar(512)` PK | 原始完整 URL（含 SSO Token 或參數） |
| **ShortURL** | `nvarchar(50)` | Picsee 產生的短網址（如 `https://picsee.io/xxxxx`） |

---

## 主鍵

```
PRIMARY KEY (URL)
```

---

## 備註

- URL 長度上限 512 字元，支援含長參數的 SSO 連結。
- 短網址一經產生即永久快取，不會過期（除非手動清除此表）。
- Oracle 資料庫的欄位名為大寫 `SHORTURL`，程式碼中做了相容處理（`rows[0]["ShortURL"]` 或 `rows[0]["SHORTURL"]`）。
- GetPicsUrl 若 API 呼叫失敗，會回傳原始 URL（降級處理，不寫入此表）。
- 與 SYS_URL_LIST 為不同維度的短網址映射，SYS_URL_LIST2 是實際使用的表。
