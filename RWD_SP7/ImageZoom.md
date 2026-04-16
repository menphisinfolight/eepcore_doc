# ImageZoom

> `EEPRWDTools.Core/Controls/ImageZoom.cs` — 60 行
> 繼承：`RWDControl` → `Component`

## 用途

**圖片放大鏡元件**。提供圖片縮放檢視功能，支援多種放大模式（標準、反轉、拖曳、內部放大）。可指定放大區域的位置和尺寸，適用於產品細節展示。

## JSON 設定範例

```json
{
  "type": "imagezoom",
  "id": "imgZoomProduct",
  "remoteName": "cmdProductImages",
  "imageField": "SMALL_IMAGE",
  "largeImageField": "LARGE_IMAGE",
  "position": "right",
  "zoomType": "standard",
  "imageWidth": "400px",
  "zoomWidth": "300px"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源 |
| **pageSize** | int | 數字框 `[NumberboxEditor]`（min:1） | `5` | 每頁顯示筆數 |
| **imageFolder** | string | 文字 | — | 圖片資料夾路徑 |
| **imageField** | string | 欄位選擇器 `[ColumnEditor]` | — | 小圖欄位名稱 |
| **largeImageField** | string | 欄位選擇器 `[ColumnEditor]` | — | 大圖欄位名稱 |
| **images** | List\<Image\> | 集合編輯器 `[CollectionEditor]` | `[]` | 靜態圖片集合 |
| **position** | string | 選項 `[ItemsEditor]` | `"right"` | 放大區域位置 |
| **zoomType** | string | 選項 `[ItemsEditor]` | `"standard"` | 放大模式 |
| **imageWidth** | string | 文字 | — | 圖片寬度 |
| **title** | string | 文字 | — | 標題 |
| **zoomWidth** | string | 文字 | — | 放大區域寬度 |
| **onBeforeLoad** | string | 事件編輯器 `[ScriptEditor]`（param） | — | 載入前事件 |

### Position 可選值

`left`、`right`、`top`、`bottom`

### ZoomType 可選值

| 值 | 說明 |
|---|------|
| `standard` | 標準放大（滑鼠移入顯示放大區域） |
| `reverse` | 反轉模式 |
| `drag` | 拖曳放大 |
| `innerzoom` | 內部放大（在圖片本身上方顯示放大效果） |

### 靜態 Image 子項目屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **src_small** | string | 圖片選擇器 `[ImageEditor]` | 小圖路徑 |
| **src_large** | string | 圖片選擇器 `[ImageEditor]` | 大圖路徑 |

## 備註

- 渲染時輸出 `<div class="bootstrap-imagezoom">`，前端 JS 處理放大鏡邏輯。
- 支援兩種資料模式：透過 `remoteName` 綁定遠端資料，或透過 `images` 設定靜態圖片。
