# ImageList

> `EEPRWDTools.Core/Controls/ImageList.cs` — 76 行
> 繼承：`RWDControl` → `Component`

## 用途

**圖片列表元件**。以網格方式顯示多張圖片，支援遠端資料來源或靜態圖片集合。可設定分頁、每行欄數、點擊事件等。適用於產品圖庫、相簿展示等場景。

## JSON 設定範例

```json
{
  "type": "imagelist",
  "id": "imgProducts",
  "remoteName": "cmdProducts",
  "pageSize": 6,
  "pagination": true,
  "imageField": "PHOTO",
  "captionField": "PRODUCT_NAME",
  "urlField": "LINK_URL",
  "horizontalColumnsCount": 3,
  "style": "shadow"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源 |
| **pageSize** | int | 數字框 `[NumberboxEditor]`（min:1） | `5` | 每頁顯示筆數 |
| **pagination** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 是否顯示分頁 |
| **imageFolder** | string | 文字 | — | 圖片資料夾路徑 |
| **imageField** | string | 欄位選擇器 `[ColumnEditor]` | — | 圖片欄位名稱 |
| **captionField** | string | 欄位選擇器 `[ColumnEditor]` | — | 標題欄位名稱 |
| **urlField** | string | 欄位選擇器 `[ColumnEditor]` | — | 連結 URL 欄位 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（panel） | — | 綁定的 Panel 元件 |
| **images** | List\<Image\> | 集合編輯器 `[CollectionEditor]` | `[]` | 靜態圖片集合 |
| **horizontalColumnsCount** | int | 數字框 `[NumberboxEditor]`（min:1） | `3` | 每行顯示欄數 |
| **containerCls** | string | 選項 `[ItemsEditor]` | — | 容器類別（`container` / `container-fluid`） |
| **style** | string | 選項 `[ItemsEditor]`（多選） | — | 互動效果（`shadow` / `transform`） |
| **textFormatter** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 文字格式化函式 |
| **onClick** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 點擊事件函式 |

### 靜態 Image 子項目屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **src** | string | 圖片選擇器 `[ImageEditor]` | 圖片路徑 |
| **caption** | string | 文字 | 圖片標題 |
| **url** | bool | 選單連結 `[MenuEditor]` | 點擊連結目標 |

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 8478–8702 行
> jQuery 外掛名稱：`$.fn.imagelist`

### 初始化流程

1. 解析 `data-options`，先呼叫 `bindEvent` 綁定分頁事件，再呼叫 `load` 載入資料。

### 資料載入（load）

- **遠端模式**（`remoteName` 有值）：設定 `imageUrl = '../file'`，先以 `fadeOut(500)` 淡出，再透過 `$.loadData()` 查詢（傳入 `rows`=pageSize、`page`、`whereStr`、`whereItems`、`total:true`），回傳後呼叫 `loadData` 渲染並以 `fadeIn(500)` 淡入。
- **靜態模式**（`images` 有值）：設定 `imageUrl = '../file/images'`，欄位名稱固定為 `src`、`caption`、`url`，`pagination` 強制為 false。

### 渲染邏輯（loadData）

1. 清空容器內容，依據 `horizontalColumnsCount` 計算 Bootstrap 欄位 class（`col-sm-{12/count}`）。
2. 欄數上限為 6；若設定 5 則自動調整為 4。
3. 每 `horizontalColumnsCount` 筆圖片包裹為一個 `<div class="row">`。
4. 每筆圖片渲染 `<div class="image-box {style}">` 內含 `<img class="img-responsive image-item">`。
5. 若有 `captionField` 或 `textFormatter`，在圖片下方顯示文字。`textFormatter` 為自訂函式，接收 `row` 參數。
6. 所有文字輸出皆經過 `$.validateScript()` 防止 XSS。

### 點擊行為

- **圖片區塊（`.image-box`）點擊**：若有 `urlField`，http(s) 開頭的連結以新分頁開啟，否則載入至 `bindingObject` 指定的 Panel。
- **圖片/文字（`.image-item`, `.image-text`）點擊**：若有 `onClick` 回呼，傳入對應的 `row` 資料物件。

### 分頁控制（refreshPagination）

- 當 `pagination` 為 true 時，動態產生分頁 UI（`<ul class="pagination">`），包含：重新整理、首頁、上一頁、頁碼（前後各 2 頁）、下一頁、末頁。
- 當前頁碼標記 `active`，首尾頁時停用前/後按鈕（`disabled`）。
- 若有 `$.fn.locale.pageCount`，顯示總頁數資訊。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化、綁定事件、載入資料 |
| `bindEvent` | 綁定分頁按鈕點擊事件 |
| `load` | 載入資料（支援 `page` 參數指定頁碼） |
| `loadData` | 接收資料物件，渲染圖片網格與分頁 |
| `refreshPagination` | 根據 total 重新產生分頁控制列 |
| `setWhere` | 設定查詢條件（`whereStr`），重新載入 |

## 備註

- ContainerCls 標記為 `[DataOption(false)]`，不輸出到 data-options，而是用於伺服端渲染的外層容器 CSS 類別。
- 支援兩種資料模式：透過 `remoteName` 綁定遠端資料，或透過 `images` 設定靜態圖片。
- 渲染時包含分頁元素 `<ul class="pagination">`，由前端 JS 控制分頁邏輯。
