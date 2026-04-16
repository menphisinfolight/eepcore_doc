# HtmlString

> `EEPRWDTools.Core/HtmlString.cs` — 144 行

## 用途

**HTML 產生工具類別**（HTML Builder Utility）。

HtmlString 是一個輕量的 HTML 字串建構器，透過堆疊（Stack）追蹤巢狀標籤，並以縮排方式產生格式化的 HTML 輸出。所有 RWD 控制項的 `Render()` 方法皆透過此類別組裝 HTML。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Strings` | List\<string\> | 儲存所有已組裝的 HTML 行 |
| `Tags` | Stack\<string\> | 追蹤尚未關閉的標籤名稱（用於 `AppendEndTag` 自動關閉） |

## 核心方法

| 方法 | 說明 |
|------|------|
| `Append(string value)` | 加入一行文字，依據 `Tags` 深度自動縮排（每層 2 個空白） |
| `AppendTag(tag, tagClass, content, attrs)` | 產生完整標籤。若 `content` 不為 null → 自閉合（開標籤＋內容＋關標籤同一行）；若為 null → 僅開標籤並 Push 至 `Tags` |
| `AppendBeginTag(tag, tagClass, attrs)` | 產生開標籤並 Push（等同 `AppendTag` 不帶 content） |
| `AppendEndTag()` | 從 `Tags` Pop 標籤名稱，產生對應的關閉標籤 |
| `AppendMeta(attr)` | 產生 `<meta>` 標籤 |
| `AppendScript(src)` | 產生 `<script src="..."></script>` |
| `AppendScripts(srcs)` | 批次加入多個 script 標籤 |
| `AppendStyleSheet(href, id)` | 產生 `<link rel="stylesheet">` 標籤 |
| `AppendStyleSheets(hrefs)` | 批次加入多個 stylesheet 標籤 |
| `GetAtributes(attrs)` | 將屬性物件轉為 `Dictionary<string, string>`（支援三種輸入） |
| `ToString()` | 以 `\r\n` 串接所有行，輸出最終 HTML 字串 |

## GetAtributes 屬性轉換邏輯

`attrs` 參數支援三種型別：

| 輸入型別 | 轉換方式 |
|----------|----------|
| `string` | 視為 `data-options` 值 → `{ "data-options": value }` |
| `IDictionary<string, string>` | 直接使用 |
| 匿名物件 / 一般物件 | 以反射取得所有 Property → 轉為 Dictionary |

## 備註

- `link` 和 `meta` 標籤為自閉合標籤，`AppendTag` 中有特殊判斷：給定 content 時不輸出關閉標籤。
- 標籤的巢狀深度由 `Tags.Count` 決定縮排量，確保產出的 HTML 可讀性。
- 此類別僅負責字串組裝，不做任何 HTML 編碼或跳脫處理。
