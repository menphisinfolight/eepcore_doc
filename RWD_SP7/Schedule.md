# Schedule

> `EEPRWDTools.Core/Controls/Schedule.cs` — 96 行
> 繼承：`RWDControl` → `Component`

## 用途

**行事曆元件**（Schedule）。

Schedule 提供年/月/週/日的行事曆檢視，可綁定資料來源顯示事件項目。支援時間區段設定、事件編輯、新增，以及自訂渲染與格式化。

## JSON 設定範例

```json
{
  "type": "schedule",
  "id": "schCalendar",
  "remoteName": "cmdEvent",
  "titleField": "EVENT_TITLE",
  "textField": "EVENT_TEXT",
  "dateField": "EVENT_DATE",
  "dateToField": "EVENT_DATE_TO",
  "dateFormat": "datetime",
  "timeFromField": "TIME_FROM",
  "timeToField": "TIME_TO",
  "timeFormat": "HH:mm",
  "views": "month,week,day",
  "defaultView": "month",
  "defaultItemClass": "event-info",
  "firstDay": "2",
  "startHour": 8,
  "endHour": 20,
  "timeSplit": "60",
  "editable": true,
  "allowAdd": false
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源命令 |
| **titleField** | string | 欄位選擇器 `[ColumnEditor]` | — | 事件標題欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 事件內文欄位 |
| **dateField** | string | 欄位選擇器 `[ColumnEditor]` | — | 事件起始日期欄位 |
| **dateToField** | string | 欄位選擇器 `[ColumnEditor]` | — | 事件結束日期欄位 |
| **dateFormat** | string | 下拉選擇 `[ItemsEditor]` | `"datetime"` | 日期格式（datetime / varchar8） |
| **timeFromField** | string | 欄位選擇器 `[ColumnEditor]` | — | 起始時間欄位 |
| **timeToField** | string | 欄位選擇器 `[ColumnEditor]` | — | 結束時間欄位 |
| **timeFormat** | string | 下拉選擇 `[ItemsEditor]` | `"HH:mm"` | 時間格式（HH:mm / HHmm） |
| **views** | string | 多選 `[ItemsEditor]` | `"month,week,day"` | 可用檢視模式（year / month / week / day） |
| **defaultView** | string | 下拉選擇 `[ItemsEditor]` | `"month"` | 預設檢視模式 |
| **defaultItemClass** | string | 下拉選擇 `[ItemsEditor]` | `"event-info"` | 預設事件樣式（event-important / event-warning / event-info / event-inverse / event-success / event-special） |
| **firstDay** | string | 下拉選擇 `[ItemsEditor]` | `"2"` | 每週起始日（2=Sunday / 1=Monday） |
| **startHour** | int | 數字框 `[NumberboxEditor]` | 8 | 日/週檢視的起始小時 |
| **endHour** | int | 數字框 `[NumberboxEditor]` | 20 | 日/週檢視的結束小時 |
| **timeSplit** | string | 下拉選擇 `[ItemsEditor]` | `"60"` | 時間間隔分鐘（15 / 30 / 60） |
| **monthTitle** | bool | 勾選 `[CheckboxEditor]` | `false` | 月檢視是否顯示標題 |
| **editable** | bool | 勾選 `[CheckboxEditor]` | `true` | 是否可編輯事件 |
| **allowAdd** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否允許新增事件 |
| **editForm** | string | 元件選擇器 `[ControlEditor]` | — | 編輯用的 DataForm 元件 ID |
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 載入前事件，簽名 `function(param){}` |
| **onRenderItem** | string | 腳本編輯器 `[ScriptEditor]` | — | 事件項目渲染時觸發，簽名 `function(event){}` |
| **onTimeFormat** | string | 腳本編輯器 `[ScriptEditor]` | — | 自訂時間格式化，簽名 `function(value){ return value; }` |
| **onInsert** | string | 腳本編輯器 `[ScriptEditor]` | — | 新增事件時觸發，簽名 `function(row){}` |
| **onUpdate** | string | 腳本編輯器 `[ScriptEditor]` | — | 更新事件時觸發，簽名 `function(row){}` |

## 前端使用範例

```javascript
// OnRenderItem — 自訂行事曆項目顯示內容（附加部門欄位）
function Schedule1_onRenderItem(event) {
    event.text += event.row["部門"];
}

// OnBeforeLoad — 動態過濾條件（只顯示當前使用者的排程）
function Schedule1_onBeforeLoad(data) {
    data.whereItems = JSON.stringify([{
        field: 'USERID',
        operator: '=',
        value: $.fn.clientInfo.user
    }]);
}

// OnInsert — 新增事件前設定預設值
function Schedule1_onInsert(row) {
    row.USERID = $.fn.clientInfo.user;
}
```

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 14108–14619 行
> jQuery 插件名稱：`$.fn.schedule`（透過 `$.createObj('schedule', ...)` 建立）

### HTML 結構

```
div.schedule-container          ← init 時自動用 wrap 包裹
  div.schedule-header           ← createHeader 產生的工具列
    div.pull-right.form-inline
      div.btn-group             ← [+] / << / 今天 / >> 導覽按鈕
      div.btn-group             ← month / week / day 檢視切換按鈕
    h4                          ← 顯示目前年月標題（onAfterViewLoad 更新）
  div.bootstrap-schedule        ← 原始元件 div，交由 bootstrap-calendar 渲染
div.modal.fade#{id}_modal       ← createModal 動態附加至 body（非 editForm 模式）
```

### 公開 API 方法

| 方法 | 簽名 | 說明 |
|------|------|------|
| `init` | `(jq, options)` | 初始化：建立 header、modal、實例化 `$.fn.calendar`，綁定事件後自動呼叫 `load` |
| `options` | `(jq)` → object | 取得元件設定物件 |
| `load` | `(jq, force?)` | 根據目前檢視日期範圍組合 whereItems，透過 `$.loadData` 向後端查詢事件資料 |
| `loadData` | `(jq, data)` | 將事件陣列設入 calendar 的 `events_source`，重新渲染檢視 |
| `createHeader` | `(jq)` | 產生工具列 HTML（導覽按鈕 + 檢視切換按鈕），插入元件前方 |
| `createModal` | `(jq)` | 產生預設 modal HTML（header / body / footer），附加至 `body` |
| `bindEvent` | `(jq)` | 綁定 header 上的 `data-calendar-nav`（prev/today/next）與 `data-calendar-view` 點擊事件 |
| `insert_row` | `(jq)` | 開啟 modal 進入新增模式：清除 event-id、觸發 `onInsert` 回呼 |
| `delete_row` | `(jq)` | 刪除目前選取事件：彈出確認對話框後呼叫 `$.updateData` 送出 deleted 陣列 |
| `submit` | `(jq)` | 儲存 modal 表單：從 modal 的 `[name="title"]`、`[name="start"]`、`[name="end"]` 取值，判斷 insert/update 後呼叫 `$.updateData` |
| `setValue` | `(jq, value)` | 設定日期值（內部使用 datebox 格式轉換） |
| `setWhere` | `(jq, where)` | 設定查詢條件（支援陣列或字串），設定後自動重新 `load` |

### 關鍵行為

1. **資料載入與分頁**（`load`，第 14335–14451 行）
   - 依照目前 calendar 的 `position.start` 計算當月起迄日期（含前後跨週天數）。
   - 若 `view` 為 `year`，則範圍擴展為整年（1/1–12/31）。
   - 以 `dateField` 與 `dateToField` 組合 `>=` / `<=` 條件加入 whereItems。
   - 使用 `opts.monthStart` 快取避免同月重複查詢；`force=true` 時強制重新載入。
   - 查詢結果的每筆 row 轉換為 calendar event 物件（含 `id`、`title`、`text`、`class`、`start`、`end` 時間戳）。
   - 若事件起訖時間相同且為整日（午夜），自動將 `end` 延長至當日 23:59:59。

2. **Modal 編輯流程**（第 14160–14246 行）
   - `editForm` 模式：modal 即為指定的 DataForm，透過 `form('loadRow')` / `form('loadDetail')` 載入資料，狀態為 `inserted` 或 `updated`。
   - 模板模式：使用 bootstrap-calendar 內建 modal 模板，內含 `[name="title"]`、`[name="text"]`、`[name="start"]`、`[name="end"]` 欄位，搭配 datebox 控制項。
   - `editable=true` 時動態插入「儲存」與「刪除」按鈕至 `modal-footer`。
   - `editable=false` 時禁用 modal 內所有 input/textarea/button。

3. **日期格式處理**（第 14395–14412 行）
   - `dateFormat` 為 `varchar8` 時，8 碼字串（`yyyyMMdd`）會先轉為標準日期再取 timestamp。
   - `timeFormat` 為 `HH:mm` 時以冒號分隔，否則為 4 碼數字（`HHmm`）。

4. **submit 儲存邏輯**（第 14547–14603 行）
   - 從 modal 讀取 start/end 值，依 `dateFormat` 與 `timeFormat` 格式化回寫 row。
   - 根據是否存在 event（update or insert）組裝 `datas` 陣列，呼叫 `$.updateData` 送出。
   - 成功後關閉 modal 並以 `force=true` 重新 `load`。

5. **依賴套件**
   - 使用 `bootstrap-calendar`（`$.fn.calendar`）作為底層日曆渲染引擎，模板路徑由 `defaults.tmpl_path` 指定。
   - 多語系透過 `window.calendar_languages` 取得。
   - 使用 `lodash` 的 `_.find` 查找事件。

## 備註

- Render 輸出 `<div class="bootstrap-schedule">`，前端由 JavaScript 行事曆套件驅動。
- `dateFormat` 為 `"varchar8"` 時，日期欄位以 8 碼字串格式（如 `20260416`）儲存。
- `editForm` 僅接受 `dataform` 類型元件。
