# Css

> `EEPRWDTools.Core/Controls/Css.cs` — 18 行
> 繼承：`RWDControl` → `Component`

## 用途

**CSS 樣式表引用元件**。在頁面中插入一個 `<link>` 標籤，載入指定的外部 CSS 檔案。檔案透過 `../file/css?q=` 路徑取得。

## JSON 設定範例

```json
{
  "type": "css",
  "id": "cssCustom",
  "src": "custom/style.css"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **src** | string | 文字 | — | CSS 檔案路徑 |

## 備註

- Render 直接輸出 `<link href="../file/css?q={Src}" rel="stylesheet">`，不使用 RenderTag。
- 用於在模組中引入自訂 CSS 樣式。
