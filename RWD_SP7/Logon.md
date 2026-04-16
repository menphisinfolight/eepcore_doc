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

## 備註

- Render 時會先輸出一個隱藏的 `<textarea id="_PUBLICKEY">`，內含 RSA 公鑰，供前端加密密碼使用。
- 內部建立 `LogonForm` 並設定 `database`、`solution`、`encrypt`、`captcha`、`forgetPassword`、`registered` 等 data-options。
- Title、UserText、PasswordText 均標記 `[Localization("id")]`，支援依元件 ID 進行多語系翻譯。
- LogonForm 的 CSS 類別固定為 `bootstrap-logon`，水平欄數固定為 1。
