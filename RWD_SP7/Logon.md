# Logon

> `EEPRWDTools.Core/Controls/Logon.cs` — 58 行
> 繼承：`RWDControl` → `Component`

## 用途

**登入表單元件**。產生一個 Bootstrap 風格的登入表單，包含使用者帳號、密碼欄位，可選配驗證碼、忘記密碼、註冊功能。內部使用 `LogonForm` 子元件渲染，並自動處理 RSA 加密公鑰。

## JSON 設定範例

```json
{
  "type": "logon",
  "id": "frmLogon",
  "title": "系統登入",
  "userText": "帳號",
  "passwordText": "密碼",
  "captcha": true,
  "forgetPassword": false,
  "registered": false
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 `[Localization]` | — | 登入表單標題（支援多語系） |
| **userText** | string | 文字 `[Localization]` | `"user"` | 使用者欄位標籤（支援多語系） |
| **passwordText** | string | 文字 `[Localization]` | `"password"` | 密碼欄位標籤（支援多語系） |
| **captcha** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否啟用驗證碼 |
| **forgetPassword** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否顯示忘記密碼連結 |
| **registered** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否顯示註冊連結 |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.logon`（`bootstrap.infolight.js` 第 17936–18195 行）

### 初始化流程

1. 解析 `data-options` 取得設定。
2. 若 `captcha` 為 true，動態插入驗證碼欄位（`<input name="code">`）及可點擊重新整理的驗證碼圖片。
3. 若 `forgetPassword` 或 `registered` 為 true，動態插入忘記密碼／註冊帳號連結，連結指向 `../account?type=resetP` 或 `../account?type=registerU`。
4. 綁定 `.form-logon`、`.form-cancel` 按鈕及 `a.close` 事件。

### 主要方法

| 方法 | 說明 |
|------|------|
| `show(callback)` | 清空表單、重新載入驗證碼圖片，以 Bootstrap Modal 顯示登入框。可傳入成功回呼。 |
| `clear()` | 清空表單內所有 `input`、`select` 值。 |
| `getRow()` | 收集表單中 `.form-control` 的值，回傳 `{ user, password, code }` 物件。 |
| `logon()` | 執行登入：使用 **JSEncrypt** 以 `#_PUBLICKEY` 的 RSA 公鑰加密帳號與密碼，POST 至 `../logon`（`mode: 'logon'`）。成功時將 `clientInfo` 存入 `sessionStorage` 並呼叫 `onSuccess` 回呼；失敗則顯示多語系錯誤訊息（`alert-danger`）。 |
| `logout()` | POST `../logon`（`mode: 'logoff'`），清除 `sessionStorage.clientInfo` 並導回原頁面。 |
| `cancel()` | 關閉 Modal。 |
| `changePWD()` | 動態建立變更密碼 Modal（舊密碼、新密碼、確認密碼），密碼一致後以 RSA 加密 POST `../account`（`mode: 'changeP'`）。 |

### 加密機制

- 使用 `JSEncrypt` 函式庫進行 RSA 加密。
- `encryptValue()` 遞迴呼叫：若加密結果包含 `==` 則重新加密，直到取得不含 `==` 的密文。

### 事件回呼

| 回呼 | 觸發時機 |
|------|----------|
| `onSuccess(clientInfo)` | 登入成功後，參數為伺服端回傳的 clientInfo 物件 |

## 備註

- Render 時會先輸出一個隱藏的 `<textarea id="_PUBLICKEY">`，內含 RSA 公鑰，供前端加密密碼使用。
- 內部建立 `LogonForm` 並設定 `database`、`solution`、`encrypt`、`captcha`、`forgetPassword`、`registered` 等 data-options。
- Title、UserText、PasswordText 均標記 `[Localization("id")]`，支援依元件 ID 進行多語系翻譯。
- LogonForm 的 CSS 類別固定為 `bootstrap-logon`，水平欄數固定為 1。
