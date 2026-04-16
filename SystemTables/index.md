# EEP Core 系統資料表

> ⚠️ **舊版遺留（未使用）**：SYS_TODOHIS、SYS_TODOLIST、SYS_SSO_SOLUTION、SYS_INFOPAY、SYS_ANYQUERY、MENUTABLELOG、SYS_FLDEFINITION、SYS_EXTAPPROVE、SYSERRLOG
>
> 這些表在 SQL 腳本中有定義，但 C# 與 JavaScript 程式碼均無引用。

## 資料表清單

- [COLDEF](./COLDEF.md) — 欄位定義中繼資料表
- [SYSAUTONUM](./SYSAUTONUM.md) — 自動編號流水號表
- [GROUPS](./GROUPS.md) — 群組/角色主檔
- [GROUPMENUS](./GROUPMENUS.md) — 群組選單權限表
- [GROUPMENUCONTROL](./GROUPMENUCONTROL.md) — 群組選單控制項權限表（⚠️ 程式碼無引用）
- [USERS](./USERS.md) — 使用者主檔
- [USERS_LOG](./USERS_LOG.md) — 使用者登入歷程記錄表
- [USERS_LOGON](./USERS_LOGON.md) — 使用者登入驗證次數控制表
- [USERMENUS](./USERMENUS.md) — 使用者選單權限表
- [USERMENUCONTROL](./USERMENUCONTROL.md) — 使用者選單控制項權限表（⚠️ 程式碼無引用）
- [USERGROUPS](./USERGROUPS.md) — 使用者-群組關聯表
- [MENUITEMTYPE](./MENUITEMTYPE.md) — 選單方案定義表
- [MENUTABLE](./MENUTABLE.md) — 選單主檔
- [MENUTABLECONTROL](./MENUTABLECONTROL.md) — 選單控制項定義表（⚠️ 程式碼無引用）
- [MENUTABLELOG](./MENUTABLELOG.md) ⚠️ 未使用
- [MENUCHECKLOG](./MENUCHECKLOG.md) — 選單簽入/簽出記錄表
- [MENUFAVOR](./MENUFAVOR.md) — 選單我的最愛表
- [SYSEEPLOG](./SYSEEPLOG.md) — EEP 系統操作日誌表
- [SYSSQLLOG](./SYSSQLLOG.md) — SQL 執行日誌表
- [SYSERRLOG](./SYSERRLOG.md) — 系統錯誤日誌表 ⚠️ 未使用
- [SYS_LANGUAGE](./SYS_LANGUAGE.md) — 系統多語系辭典表（⚠️ C# 無直接引用）
- [SYS_MESSENGER](./SYS_MESSENGER.md) — 系統訊息傳遞表
- [SYS_REFVAL](./SYS_REFVAL.md) — 下拉選單參照定義表
- [SYS_REFVAL_D1](./SYS_REFVAL_D1.md) — 下拉選單參照明細表
- [SYS_ANYQUERY](./SYS_ANYQUERY.md) ⚠️ 未使用
- [SYS_REPORT](./SYS_REPORT.md) — 報表定義表（⚠️ C# 無直接引用表名）
- [SYS_PERSONAL](./SYS_PERSONAL.md) — 使用者個人化設定表（⚠️ C# 無直接引用）
- [SYS_ORG](./SYS_ORG.md) — 組織架構主檔
- [SYS_ORGKIND](./SYS_ORGKIND.md) — 組織類型定義表
- [SYS_ORGLEVEL](./SYS_ORGLEVEL.md) — 組織層級定義表
- [SYS_ORGROLES](./SYS_ORGROLES.md) — 組織角色關聯表
- [SYS_ROLES_AGENT](./SYS_ROLES_AGENT.md) — 角色代理設定表
- [SYS_USERS_AGENT](./SYS_USERS_AGENT.md) — 使用者代理設定表
- [SYS_TODOHIS](./SYS_TODOHIS.md) ⚠️ 未使用
- [SYS_TODOLIST](./SYS_TODOLIST.md) ⚠️ 未使用
- [SYS_FLDEFINITION](./SYS_FLDEFINITION.md) ⚠️ 未使用
- [SYS_FLINSTANCESTATE](./SYS_FLINSTANCESTATE.md) — 流程實例狀態表（⚠️ 舊版遺留）
- [SYS_EXTAPPROVE](./SYS_EXTAPPROVE.md) ⚠️ 未使用
- [USERDEVICES](./USERDEVICES.md) — 使用者裝置註冊表（MAUI APP 模組專用）
- [SYS_SSO_SOLUTION](./SYS_SSO_SOLUTION.md) ⚠️ 未使用
- [SYS_DOCFILES](./SYS_DOCFILES.md) — Word 報表定義表
- [SYS_XLSFILES](./SYS_XLSFILES.md) — Excel 報表定義表
- [SYS_TRSFILES](./SYS_TRSFILES.md) — 資料表交易回寫定義表
- [SYS_PARAS](./SYS_PARAS.md) — 系統參數表
- [FLOWINSTANCE](./FLOWINSTANCE.md) — 流程實例表
- [FLOWTODO](./FLOWTODO.md) — 流程待辦事項表
- [FLOWNOTIFY](./FLOWNOTIFY.md) — 流程通知表
- [FLOWHISTORY](./FLOWHISTORY.md) — 流程簽核歷史表
- [FLOWCOMMENT](./FLOWCOMMENT.md) — 流程評論表
- [FLOWCOMMENTDETAIL](./FLOWCOMMENTDETAIL.md) — 流程評論接收者明細表
- [FLOWLANGUAGE](./FLOWLANGUAGE.md) — 流程多語系名稱表
- [SYS_CARDS](./SYS_CARDS.md) — 首頁卡片定義表
- [USERCARDS](./USERCARDS.md) — 使用者卡片權限表
- [GROUPCARDS](./GROUPCARDS.md) — 群組卡片權限表
- [SYS_URL_LIST](./SYS_URL_LIST.md) — 短網址對應表（DB2/Informix 使用）
- [SYS_URL_LIST2](./SYS_URL_LIST2.md) — 短網址快取表
- [SYS_SCHEDULE](./SYS_SCHEDULE.md) — 排程任務定義表
- [SYS_SCHEDULE_LOG](./SYS_SCHEDULE_LOG.md) — 排程任務執行日誌表
- [SYS_NOTIFY](./SYS_NOTIFY.md) — 系統公告表（⚠️ NotifyProvider 未完整實作）
- [SYS_NOTIFY_DETAIL](./SYS_NOTIFY_DETAIL.md) — 系統公告接收者明細表（⚠️ 程式碼無引用）
- [SYS_USERDEF](./SYS_USERDEF.md) — 使用者自訂報表定義表
- [SYS_INFOPAY](./SYS_INFOPAY.md) ⚠️ 未使用
- [FLOWPHRASE](./FLOWPHRASE.md) — 流程常用片語表
- [SYS_GPTLOG](./SYS_GPTLOG.md) — GPT AI 使用日誌表（⚠️ 僅 Oracle 腳本）
