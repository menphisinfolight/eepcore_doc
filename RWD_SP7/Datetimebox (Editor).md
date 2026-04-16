# Datetimebox (Editor)

> `EEPRWDTools.Core/Editors/Datetimebox.cs` — 45 行
> 繼承：`RWDEditor`

## 用途

**日期時間選取編輯器**。結合日期和時間選取功能，Render 時將日期格式和時間格式合併為完整的 datetime 格式。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **format** | string | 下拉選項 `[ItemsEditor]` | "yyyy-mm-dd" | 日期格式 |
| **timeFormat** | string | 下拉選項 `[ItemsEditor]` | "hh:ii" | 時間格式（hh:ii:ss / hh:ii / hhiiss / hhii） |
| **dataType** | DateboxDataType | 列舉 | — | 資料類型 |
| **selectOnly** | bool | 核取方塊 | false | 是否僅能從日曆選取 |
| **pickerPosition** | string | 下拉選項 `[ItemsEditor]` | — | 選取器位置 |
| **prompt** | string | 文字 | — | placeholder 提示文字 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(date)]` | — | 選取事件 |
| **minView** | DatetimeboxView | 列舉 | — | 最小檢視層級 |

## 備註

- Render 時 Format 會被合併為 `"{日期格式} {時間格式}"`（如 `"yyyy-mm-dd hh:ii"`）。
- 共用 Datebox 的 `bootstrap-datebox` CSS 類別。
