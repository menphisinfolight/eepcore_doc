# Datebox (Editor)

> `EEPRWDTools.Core/Editors/Datebox.cs` — 39 行
> 繼承：`RWDEditor`

## 用途

**日期選取編輯器**。提供日期選取器（Date Picker），支援多種日期格式和選取器位置設定。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 下拉選項 `[ItemsEditor]` | "yyyy-mm-dd" | 日期格式 |
| **dataType** | DateboxDataType | 列舉 | — | 資料類型 |
| **selectOnly** | bool | 核取方塊 | false | 是否僅能從日曆選取 |
| **pickerPosition** | string | 下拉選項 `[ItemsEditor]` | — | 選取器位置（top-left / top-right / bottom-left / bottom-right） |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(date)]` | — | 選取事件 |

## 備註

- 支援格式：`yyyy/mm/dd`、`yyyy-mm-dd`、`yyyy.mm.dd`、`dd/mm/yyyy`、`dd-mm-yyyy`、`mm/dd/yyyy`、`mm-dd/yyyy`。
