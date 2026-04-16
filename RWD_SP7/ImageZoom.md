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

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 8933–9063 行
> jQuery 外掛名稱：`$.fn.imagezoom`

### 初始化流程

1. 解析 `data-options`，呼叫 `load` 方法載入資料。

### 資料載入（load）

- **遠端模式**（`remoteName` 有值）：設定 `imageUrl = '../file'`。載入前若有 `onBeforeLoad` 回呼，先呼叫之（傳入查詢參數物件，可於此修改 `whereStr` 等）。透過 `$.loadData()` 查詢後呼叫 `loadData`。
- **靜態模式**（`images` 有值）：設定 `imageUrl = '../file/images'`，欄位名稱固定為 `src_small`（小圖）/ `src_large`（大圖）。

### 渲染邏輯（loadData）

1. 清空容器，以第一筆資料的大圖建立主圖 `<a class="jqzoom">`，內含小圖 `<img>`。
2. 建立縮圖滑動列（Thumbelina 外掛）：
   - 所有圖片以 `<ul class="thumbelina">` 列出，每張縮圖高度 78px。
   - 左右箭頭控制滑動，設定 `maxSpeed: 140`、`easing: 2`。
3. 初始化 jqzoom 外掛：
   - `zoomType`：由屬性指定（standard / reverse / drag / innerzoom）。
   - `position`：放大區域位置。
   - `lens: true`（啟用鏡頭效果）。
   - `preloadImages: false`（不預載圖片）。
   - `zoomWidth` / `zoomHeight`：皆使用 `zoomWidth` 屬性值（注意：高度也使用寬度值，為正方形放大區域）。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化並載入資料 |
| `load` | 依資料來源模式載入（支援 `onBeforeLoad` 回呼） |
| `loadData` | 渲染主圖、縮圖滑動列並初始化 jqzoom |
| `setWhere` | 設定查詢條件（`whereStr`），重新載入 |

## 備註

- 渲染時輸出 `<div class="bootstrap-imagezoom">`，前端 JS 處理放大鏡邏輯。
- 支援兩種資料模式：透過 `remoteName` 綁定遠端資料，或透過 `images` 設定靜態圖片。
