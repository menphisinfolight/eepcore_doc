# Image

> `EEPRWDTools.Core/Controls/Image.cs` — 40 行
> 繼承：`RWDControl` → `Component`

## 用途

**圖片元件**。以 `<img>` 標籤渲染單張圖片，圖片來源透過 `../file/images?q=` 路徑載入。支援 Bootstrap 圖片樣式類別及滑鼠互動效果。

## JSON 設定範例

```json
{
  "type": "image",
  "id": "imgBanner",
  "src": "banner.jpg",
  "imageCls": "img-responsive",
  "style": "shadow"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **src** | string | 圖片選擇器 `[ImageEditor]` | — | 圖片檔案路徑 |
| **imageCls** | string | 選項 `[ItemsEditor]`（多選） | `"img-responsive"` | Bootstrap 圖片類別 |
| **style** | string | 選項 `[ItemsEditor]`（多選） | `"shadow"` | 互動效果樣式 |
| **onClick** | string | 事件編輯器 `[ScriptEditor]` | — | 點擊事件函式名稱 |

### ImageCls 可選值

`img-rounded`、`img-circle`、`img-thumbnail`、`img-responsive`

### Style 可選值

`shadow`（陰影）、`transform`（滑鼠移入變形）

## 備註

- Render 輸出的 CSS 類別由 `imageCls`、`bootstrap-image`、`style` 三者組合。
- OnClick 標記為 `[DataOption(false)]`，直接寫入 HTML `onclick` 屬性。
