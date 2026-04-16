# Literal

> `EEPRWDTools.Core/Controls/Literal.cs` — 18 行
> 繼承：`RWDControl` → `Component`

## 用途

**HTML 文字元件**。直接將自訂 HTML 內容輸出到頁面，不包裹任何外層標籤。適用於嵌入任意 HTML 片段。

## JSON 設定範例

```json
{
  "type": "literal",
  "id": "litContent",
  "html": "<div class='custom'>自訂內容</div>"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **html** | string | HTML 編輯器 `[HtmlEditor("bootstrap")]` | — | 要輸出的 HTML 內容 |

## 備註

- Render 時直接 `html.Append(Html)`，不產生外層容器標籤。
- 適合放置純 HTML 片段、嵌入第三方 Widget 等。
