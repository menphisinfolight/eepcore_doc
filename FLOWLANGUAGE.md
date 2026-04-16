# FLOWLANGUAGE

## 用途

**流程多語系名稱表**（Flow Language / Activity Text Translation）。

FLOWLANGUAGE 儲存流程及其各步驟的多語系翻譯名稱，支援 8 種語系。流程設計師在流程設計器中透過「Multilanguage Editor」對話框管理翻譯，資料以「先全刪再全插」的方式儲存。

### 使用場景

| 場景 | 說明 |
|------|------|
| **流程設計器 — 載入翻譯** | 前端 `showLanguageDialog()` 透過 `flowLanguage` infocommand 以 FlowID 為條件載入該流程的所有翻譯 |
| **流程設計器 — 重新整理** | `getLanguageTexts()` 掃描設計面板上所有活動節點，將新增的活動自動補入翻譯列表（預設值 = 活動名稱） |
| **流程設計器 — 儲存翻譯** | 儲存時以 `deleted: [{FlowID}]` + `inserted: [所有翻譯行]` 的方式先刪後插 |
| **後端刪除處理** | `ucFlowLanguage_onBeforeApply` 攔截 deleted 事件，產生 `DELETE FROM FlowLanguage WHERE FlowID = ...` |

### 語系對應

| 欄位 | 對應語系 |
|------|----------|
| ActivityText | 預設語系（流程定義中的原始名稱） |
| ActivityText1 | English |
| ActivityText2 | 中文（繁體） |
| ActivityText3 | 中文（簡體） |
| ActivityText4 | 中文（香港） |
| ActivityText5 | 日本語 |
| ActivityText6 | 한국어 |
| ActivityText7 | Other1（自訂） |
| ActivityText8 | Other2（自訂） |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.flowdesigner.js` | 前端：`showLanguageDialog()`（載入/編輯/儲存翻譯）、`getLanguageTexts()`（掃描活動節點取得翻譯清單） |
| `EEPWebClient.Core/design/server/SystemTable.json` | infocommand `flowLanguage` 定義：`SELECT * FROM FlowLanguage`，keys: `FlowID,ActivityID` |
| `SystemTable.Core/UserModule.cs` | `ucFlowLanguage_onBeforeApply()`：攔截刪除事件，產生 DELETE SQL |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **FlowID** | `nvarchar(50)` PK | NOT NULL | 流程定義編號 |
| **ActivityText** | `nvarchar(50)` PK | NOT NULL | 預設語系的步驟/流程名稱（空字串代表流程本身） |
| **ActivityText1** | `nvarchar(50)` | NULL | English 翻譯 |
| **ActivityText2** | `nvarchar(50)` | NULL | 中文（繁體）翻譯 |
| **ActivityText3** | `nvarchar(50)` | NULL | 中文（簡體）翻譯 |
| **ActivityText4** | `nvarchar(50)` | NULL | 中文（香港）翻譯 |
| **ActivityText5** | `nvarchar(50)` | NULL | 日本語翻譯 |
| **ActivityText6** | `nvarchar(50)` | NULL | 한국어 翻譯 |
| **ActivityText7** | `nvarchar(50)` | NULL | 自訂語系 1 翻譯 |
| **ActivityText8** | `nvarchar(50)` | NULL | 自訂語系 2 翻譯 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2(50)` 取代 `nvarchar(50)`。無特殊型別差異。

---

## 主鍵

```
PRIMARY KEY (FlowID, ActivityText)
```

---

## 資料生命週期

```
載入：前端 showLanguageDialog()
      → AJAX GET → infocommand flowLanguage
      → SELECT * FROM FlowLanguage WHERE FlowID = @flowID

重新整理：前端 getLanguageTexts()
      → 掃描設計面板上的 StartActivity / StandActivity / ApproveActivity / NotifyActivity / MultiApproveActivity / SeqApproveActivity / ValidateActivity
      → 將不存在於表中的新活動加入翻譯列表，ActivityText1~8 預設填入活動名稱

儲存：前端 save
      → AJAX POST → mode=save, command=flowLanguage
      → datas = [{ table: 'flowLanguage', deleted: [{FlowID}], inserted: [所有行] }]
      → 後端 ucFlowLanguage_onBeforeApply() 處理 deleted → DELETE FROM FlowLanguage WHERE FlowID = @FlowID
      → UpdateComponent 自動 INSERT 所有 inserted 行

刪除：隨流程定義刪除時，透過相同的 onBeforeApply 機制刪除
```

---

## 備註

- ActivityText 為空字串時代表流程本身的翻譯（非活動翻譯），前端顯示為「Flow: {FlowID}」。
- 儲存採「先全刪再全插」策略（DELETE WHERE FlowID → INSERT 所有行），而非逐筆 UPDATE。
- EEP.Flow 引擎本身（C#）不直接讀取此表，翻譯的套用由前端或其他顯示層處理。
- SystemTable.json 中 keys 定義為 `FlowID,ActivityID`（注意：實際欄位名為 ActivityText，此處 ActivityID 為 infocommand 的邏輯 key 對應）。
- 各資料庫版本的 SystemTable JSON（sql/oracle/mysql/informix）皆有對應的 flowLanguage infocommand 定義。
