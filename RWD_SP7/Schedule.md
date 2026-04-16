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

## 備註

- Render 輸出 `<div class="bootstrap-schedule">`，前端由 JavaScript 行事曆套件驅動。
- `dateFormat` 為 `"varchar8"` 時，日期欄位以 8 碼字串格式（如 `20260416`）儲存。
- `editForm` 僅接受 `dataform` 類型元件。
