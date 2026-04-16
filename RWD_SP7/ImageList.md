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

## 備註

- ContainerCls 標記為 `[DataOption(false)]`，不輸出到 data-options，而是用於伺服端渲染的外層容器 CSS 類別。
- 支援兩種資料模式：透過 `remoteName` 綁定遠端資料，或透過 `images` 設定靜態圖片。
- 渲染時包含分頁元素 `<ul class="pagination">`，由前端 JS 控制分頁邏輯。
