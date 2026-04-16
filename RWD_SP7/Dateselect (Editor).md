# Dateselect (Editor)

> C# 端：`EEPRWDTools.Core/Editors/Dateselect.cs` — 36 行
> 前端：`bootstrap.infolight.js` — `$.fn.dateselect`（line 15324–15759，435 行）
> 繼承：`RWDEditor`

## 用途

**日期下拉選取編輯器**。以年、月、日三個獨立 `<select>` 下拉選單選取日期。C# 端只渲染一個 `<input class="bootstrap-dateselect">`，前端 JS 再根據 `format` 動態轉換為多個 `<select>`。

**不是包第三方 date picker 套件**，而是專案內自行實作的 jQuery 元件（`$.fn.dateselect = $.createObj('dateselect', ...)`）。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 文字 | "" | 日期格式，決定顯示哪些下拉選單（見下方格式說明） |
| **yearRangeFrom** | string | 文字 | -50 | 年份範圍起始（相對當前年份的偏移量） |
| **yearRangeTo** | string | 文字 | +10 | 年份範圍結束（相對當前年份的偏移量） |
| **year** | string | 文字 | 當前年 | 年份預設值（西元） |
| **month** | string | 文字 | 當前月 | 月份預設值 |
| **day** | string | 文字 | 當前日 | 日預設值 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀（disabled 所有 select） |

## format 格式與顯示模式

format 決定產生幾個 `<select>` 以及顯示順序：

| format 範例 | 顯示模式 | 產生的下拉選單 |
|-------------|---------|--------------|
| `""` （空白） | 年月日 | 年▼ 月▼ 日▼（預設：`yyyy年mm月dd日`） |
| `yyyy/mm/dd` | 年月日 | 年▼ / 月▼ / 日▼ |
| `mm/dd/yyyy` | 年月日 | 月▼ / 日▼ / 年▼（美式順序） |
| `yyyy/mm` | 只選年月 | 年▼ / 月▼ |
| `yyyy` | 只選年 | 年▼ |
| `yyy/mm/dd` | 民國年月日 | 民國年▼ / 月▼ / 日▼ |
| `mm/dd` | 只選月日（無年） | 月▼ / 日▼ |
| `dd` | 只選日 | 日▼ |

### 格式解析邏輯（前端 JS）

```
1. 大寫轉小寫：YYYY→yyyy, MM→mm, DD→dd
2. format 為空 → 預設 "yyyy年mm月dd日"
3. 判斷顯示模式：
   - format 中沒有 mm → onlyyear = true（只選年）
   - format 中沒有 dd → onlyym = true（只選年月）
   - format 中有 yyy 但沒有 yyyy → mgn = true（民國年）
   - format 中沒有 yyyy → noyear = true（無年份）
4. 根據 yyyy/mm/dd 在 format 中的位置決定下拉選單順序
```

## 民國年支援

當 format 使用 `yyy`（三個 y）時啟用民國年模式：

- 顯示民國年（西元年 - 1911）
- `setValue` 時自動解析 2-3 位民國年（如 `107-10-18` 或 `97-10-12`）
- 2 位民國年會自動補零（如 `97` → `097`）
- 內部計算日期時會轉換回西元年（`+1911`）

## 日數自動計算

選擇年/月時，日的下拉選單會自動更新可選天數：

| 情境 | 天數計算 |
|------|---------|
| 有年有月 | `new Date(year, month, 0).getDate()`（精確計算，含閏年） |
| 無年有月 | 2月固定 29 天、4/6/9/11月 30 天、其他 31 天 |
| 切換月份 | 若原選的日超出新月份天數，自動清空 |

## 前端 API 方法

| 方法 | 說明 |
|------|------|
| `$('#field').dateselect('getValue')` | 取得格式化後的日期字串（如 `2026/04/16`） |
| `$('#field').dateselect('setValue', '2026/04/16')` | 設定日期值，自動拆分到各 select |
| `$('#field').dateselect('readonly', true)` | 設定唯讀（disabled 所有 select） |
| `$('#field').dateselect('options')` | 取得元件設定 |
| `$('#field').dateselect('loadData')` | 重新載入資料（年/月/日選項） |

### getValue 回傳格式

回傳值根據 format 組合，月/日不足兩位會補零：

```javascript
// format = "yyyy/mm/dd" → "2026/04/16"
// format = "yyyy/mm"    → "2026/04"
// format = "yyyy"       → "2026"
// format = "yyy/mm/dd"  → "115/04/16"（民國年）
```

## 渲染結構

C# 端渲染：
```html
<input class="bootstrap-dateselect" data-options="format:'yyyy/mm/dd',yearRangeFrom:'-50',yearRangeTo:'10'" />
```

前端 JS 轉換後：
```html
<div class="input-group">
  <select class="form-control form-select dateselectyear">
    <option value="1976">1976</option>
    ...
    <option value="2036">2036</option>
  </select>
  <span class="input-group-addon input-group-text">/</span>
  <select class="form-control form-select dateselectmonth">
    <option value="1">1</option>
    ...
    <option value="12">12</option>
  </select>
  <span class="input-group-addon input-group-text">/</span>
  <select class="form-control form-select dateselectday">
    <option value="1">1</option>
    ...
    <option value="30">30</option>
  </select>
  <input style="display:none" />  <!-- 原始 input 隱藏 -->
</div>
```

## 事件

| 事件 | 觸發時機 | 說明 |
|------|---------|------|
| **onSelect** | 任一 select 值改變時 | `function(value) {}` — value 為格式化後的完整日期字串 |
| （valueChanged） | 任一 select 值改變時 | 自動觸發 `$.triggerValueChanged`，通知所屬的 DataGrid/DataForm |

## 備註

- 與 Datebox 完全不同：Datebox 用日曆彈出視窗，Dateselect 用三個 `<select>` 下拉。
- format 中分隔符號（`/`、`-`、`年`、`月`、`日` 等）會自動渲染為 `<span class="input-group-addon">` 顯示在下拉選單之間。
- 若只有一個 `<select>`（如只選年），會移除 `input-group` class 避免多餘的邊框。
- `yearRangeFrom` / `yearRangeTo` 是相對值（如 `-50` 表示當前年份往前 50 年），不是絕對年份。
- 表單清空狀態（`form.status === 'clear'`）時不會載入預設值。
