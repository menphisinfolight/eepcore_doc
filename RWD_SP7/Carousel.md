# Carousel

> `EEPRWDTools.Core/Controls/Carousel.cs` — 86 行
> 繼承：`RWDControl` → `Component`

## 用途

**輪播元件**（Carousel / 幻燈片）。以 Bootstrap Carousel 元件渲染圖片輪播，支援自動播放間隔設定、左右切換控制、指示器。可綁定遠端資料來源或使用靜態圖片集合。

## JSON 設定範例

```json
{
  "type": "carousel",
  "id": "carBanner",
  "remoteName": "cmdBanners",
  "imageField": "IMAGE_PATH",
  "captionField": "TITLE",
  "urlField": "LINK_URL",
  "interval": 3000,
  "pageSize": 5
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源 |
| **pageSize** | int | 數字框 `[NumberboxEditor]`（min:1） | `5` | 最大圖片筆數 |
| **imageFolder** | string | 文字 | — | 圖片資料夾路徑 |
| **imageField** | string | 欄位選擇器 `[ColumnEditor]` | — | 圖片欄位名稱 |
| **captionField** | string | 欄位選擇器 `[ColumnEditor]` | — | 標題欄位名稱 |
| **urlField** | string | 欄位選擇器 `[ColumnEditor]` | — | 連結 URL 欄位 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（panel） | — | 綁定的 Panel 元件 |
| **images** | List\<Image\> | 集合編輯器 `[CollectionEditor]` | `[]` | 靜態圖片集合 |
| **interval** | int | 數字框 `[NumberboxEditor]`（min:1） | `5000` | 自動播放間隔（毫秒） |
| **onClick** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 點擊事件函式 |

### 靜態 Image 子項目屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **src** | string | 圖片選擇器 `[ImageEditor]` | 圖片路徑 |
| **caption** | string | 文字 | 圖片標題 |
| **url** | bool | 選單連結 `[MenuEditor]` | 點擊連結目標 |

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 8389–8476 行
> jQuery 外掛名稱：`$.fn.carousels`

### 初始化流程

1. 解析 `data-options`，立即呼叫 `load` 方法載入資料。

### 資料載入（load）

- **遠端模式**（`remoteName` 有值）：設定 `imageUrl = '../file'`，透過 `$.loadData()` 以 `remoteName` 查詢資料（傳入 `rows`=pageSize、`page`=1、`whereItems`），回傳後呼叫 `loadData` 渲染。
- **靜態模式**（`images` 有值）：設定 `imageUrl = '../file/images'`，欄位名稱固定為 `src`、`caption`、`url`，直接呼叫 `loadData`。

### 渲染邏輯（loadData）

1. 依序為每筆資料建立 `<li>` 指示器（`carousel-indicators`）與 `<div class="item">` 輪播項目。
2. 圖片 URL 格式：`{imageUrl}?q={imageField}&f={imageFolder}`。
3. 若有 `captionField`，在圖片下方顯示 `<div class="carousel-caption">`。
4. 每個 `row` 物件存入 jQuery `.data('row')`，供點擊事件取用。

### 點擊行為

- 點擊輪播項目時：
  - 若 `urlField` 的值為 `http(s)` 開頭，以 `window.open` 在新分頁開啟。
  - 若有 `bindingObject`，透過 `$.loadHtml()` 將內容載入指定 Panel。
  - 若有 `onClick` 回呼，額外觸發。

### 自動播放

初始化完畢後呼叫 `target.carousel({ interval: opts.interval })`，依據 `interval` 設定自動輪播間隔。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化並載入資料 |
| `load` | 依資料來源模式載入資料 |
| `loadData` | 接收資料陣列，渲染輪播項目 |

## 備註

- 渲染時輸出完整的 Bootstrap Carousel 結構：`carousel-indicators`、`carousel-inner`、左右 `carousel-control`。
- 使用 `data-slide="prev"` / `data-slide="next"` 控制切換方向。
- 支援兩種資料模式：透過 `remoteName` 綁定遠端資料，或透過 `images` 設定靜態圖片。
