# AccountProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/AccountProvider.cs` |
| 行數 | 1441 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `BaseProvider` |

## 用途

處理 EEP Core 的帳號管理功能，包含：使用者登入/登出、密碼重置與變更、驗證碼（Captcha / Email / SMS / APP）、SSO 單一登入、LINE 帳號綁定、Azure AD 整合、LDAP 驗證、使用者註冊與啟用，以及密碼過期策略。

## ProcessRequest 模式

| mode | 方法 | 說明 |
|------|------|------|
| `logon` | `Logon(param)` | 使用者登入（含驗證碼檢查、密碼過期判斷） |
| `getDatabases` | `GetDatabases()` | 取得可用的資料庫清單 |
| `getSolutions` | `GetSolutions()` | 取得可用的方案（Solution）清單 |
| `getSolutionVisible` | `GetSolutionVisible()` | 取得方案選擇是否可見設定 |
| `getDBVisible` | `GetDBVisible()` | 取得資料庫選擇是否可見設定 |
| `logoff` | `Logoff()` | 清除 Session，使用者登出 |
| `refreshLogon` | `RefreshLogon()` | 刷新登入狀態（回傳空字串） |
| `getCaptcha` | `GetCaptcha(param)` | 產生並發送驗證碼（Email / SMS / APP） |
| `resetP` | `ResetPWD(user, database, email)` | 密碼重置並寄送新密碼至 Email |
| `changeP` | `ChangePWD(opassword, password, clientInfo)` | 變更密碼（含強度檢查） |
| `registerU` | `RegisterUser(user, database, userName, password, email)` | 註冊新使用者 |
| `checkVerify` | `CheckVerify()` | 檢查 MAUI 授權 |
| `switch` | `SwitchUser(user)` | 管理員切換身份（需 AUTOLOGIN = 's'） |
| `getBiometric` | `GetBiometric()` | 取得生物辨識設定 |
| `getLogonUrl` | `GetLogonUrl()` | 取得登入短網址 |

## 關鍵方法

### 登入相關

| 方法 | 說明 |
|------|------|
| `LogonUser(param)` | 核心登入流程：Captcha 驗證 → USERS 查詢 → LDAP/密碼比對 → 帳號狀態檢查（停用/過期） → 建立 ClientInfo → 密碼過期警告 → 設定 Session |
| `LogonWithKey(key)` | SSO 登入：以加密 key 解析使用者資訊後自動登入 |
| `LogonLdap(domain, user, database, solution)` | LDAP/AD 域帳號登入 |
| `LogonAzure(domain, database, solution, isAPP)` | Azure AD 登入（以 Email 查詢使用者） |
| `LogonAzure2(userid, code, database, solution)` | Azure AD 第二步驗證（APP 用 azureCode） |
| `SetClientInfo(db, clientInfo)` | 登入成功後設定 ClientInfo：更新 LASTDATE/LASTTIME、載入群組/角色、解析組織角色 |

### 密碼管理

| 方法 | 說明 |
|------|------|
| `CheckCaptcha(user, code)` | 驗證碼檢查，支援 `email` / `sms` / `app` / `captcha` 模式 |
| `CheckValidateCount(config, param, clientInfo, database)` | 檢查登入失敗次數是否超過上限（pwdDate.ValidCount） |
| `AddValidateCount(config, param, msg, clientInfo, database)` | 紀錄一次登入失敗到 USERS_LOGON |
| `ClearValidateCount(config, clientInfo)` | 登入成功後清除 USERS_LOGON 紀錄 |
| `ChangePWD(opassword, password, clientInfo)` | 變更密碼：比對舊密碼 → 檢查強度（Include/Charactor） → 更新 USERS.PWD 與 PWDDATE |
| `ResetPWD(user, database, email)` | 重設密碼並 Email 通知使用者 |

### SSO 與外部整合

| 方法 | 說明 |
|------|------|
| `GetSSOKey(param)` | 登入後產生 SSO Key |
| `GetClientInfo(JObject keyValues)` | 以 SSO key 或 database/solution 建立 ClientInfo |
| `GetClientInfo(url)` | 從 Session 或 SSO 參數取得 ClientInfo，支援免登入 URL |
| `ActivateUser(p)` | 驗證並啟用透過 Email 連結註冊的使用者 |
| `SendSMS(sms, to, body)` | 透過 Twilio 發送簡訊驗證碼 |

### 組織角色

| 方法 | 說明 |
|------|------|
| `GetOrgRoles(db, roles, orgKind)` | 根據角色查詢 SYS_ORG 取得組織角色（含下屬組織遞迴） |
| `GetOrgs(db, orgNos, pOrgNos, orgKind, schemaTable)` | 遞迴查詢子組織 |

## 密碼過期策略（pwdDate）

登入時根據 `config.json` 中的 `pwdDate` 設定判斷：

| 設定項 | 說明 |
|--------|------|
| `Init` | 首次登入需強制改密碼（PWDDATE 與 LASTDATE 均為空） |
| `Valid` | 密碼有效天數，超過則回傳 `"valid"` 強制改密碼 |
| `Warning` | 到期前 N 天顯示警告 |
| `ValidCount` | 登入失敗次數上限 |
| `ValidInterval` | 失敗計數的時間區間（毫秒） |
| `Include` | 密碼需包含大小寫字母與數字 |
| `Charactor` | 密碼最少字元數 |

## 備註

- 帳號狀態由 USERS.AUTOLOGIN 控制：`"s"` = 系統管理員、`"x"` = 停用、`"X"` = 待啟用（註冊未啟用）。
- LDAP 驗證透過 `LDAPHelper.CheckUser()`，Azure AD 以 Email 欄位匹配使用者。
- LINE 綁定在登入時處理：若 `lineID` 參數不為空，將 LINE ID 寫入 USERS.LINE。
- 短網址服務使用 `pics.ee` API。
- `SwitchUser` 僅限 AUTOLOGIN = 's' 的管理員使用，切換後記錄原始使用者於 `AgentUser`。
