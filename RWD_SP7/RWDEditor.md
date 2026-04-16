# RWDEditor

> `EEPRWDTools.Core/Editors/RWDEditor.cs` — 33 行
> 繼承：實作 `IDataOption` 介面

## 用途

**Editor 基底類別**（抽象類別）。所有欄位編輯器（Textbox、Combobox、Refval 等）的共同父類別，定義 Editor 的基本結構。Editor 不是獨立元件，而是嵌入在 DataGrid / DataForm / TableGrid 的欄位中使用。

## 共用屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| **Id** | string | 元件識別碼 |
| **Name** | string | 元件名稱 |
| **DataOptionValues** | Dictionary\<string, object\> | data-options 鍵值對（IDataOption） |

## 共用方法

| 方法 | 說明 |
|------|------|
| `Render(HtmlString html)` | 抽象方法，由子類別實作 HTML 輸出 |
| `RenderTag(html, tag, tagClass, attrs)` | 產生 HTML 標籤，自動帶入 id、name、data-options |

## 備註

- RWDEditor 實作 `IDataOption` 介面，透過 `DataOptionValues` 字典產生前端 `data-options` 屬性。
- 所有子類別的屬性若標記 `[DataOption]` 會自動加入 data-options。
- `[Security("field")]` 標記的屬性可受欄位層級安全控制。
- `[MergeKey("field")]` 標記的屬性在 Grid 合併模式下以欄位為 key。
