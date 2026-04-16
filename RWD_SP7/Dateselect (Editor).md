# Dateselect (Editor)

> `EEPRWDTools.Core/Editors/Dateselect.cs` — 36 行
> 繼承：`RWDEditor`

## 用途

**日期下拉選取編輯器**。以年、月、日三個獨立下拉選單選取日期，適用於需要分別選取年月日的場景（如生日、到期日）。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 文字 | "" | 日期格式 |
| **yearRangeFrom** | string | 文字 | — | 年份範圍起始 |
| **yearRangeTo** | string | 文字 | — | 年份範圍結束 |
| **year** | string | 文字 | — | 年份預設值 |
| **month** | string | 文字 | — | 月份預設值 |
| **day** | string | 文字 | — | 日預設值 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 備註

- 與 Datebox 不同，Dateselect 不使用日曆元件，而是以三個下拉選單呈現。
