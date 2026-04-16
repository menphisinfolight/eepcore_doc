# Menu

> `EEPRWDTools.Core/Controls/Menu.cs` — 114 行
> 繼承：`RWDControl` → `Component`

## 用途

**導覽選單元件**。以 Bootstrap Navbar 渲染網站導覽列，支援多層選單（Item → SubItem）、Logo 圖片、RWD 折疊按鈕。選單項目可設定左右對齊、連結目標、點擊事件。

## JSON 設定範例

```json
{
  "type": "menu",
  "id": "navMain",
  "logo": "logo.png",
  "menuCls": "navbar-default",
  "refreshPage": false,
  "items": [
    {
      "caption": "首頁",
      "alignment": "left",
      "url": "home"
    },
    {
      "caption": "產品",
      "alignment": "left",
      "subItems": [
        { "caption": "產品A", "url": "productA" },
        { "caption": "產品B", "url": "productB" }
      ]
    },
    {
      "caption": "聯絡我們",
      "alignment": "right",
      "url": "contact"
    }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（panel） | — | 綁定的 Panel 元件 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | `[]` | 選單項目集合 |
| **menuCls** | string | 文字 | `"navbar-default"` | Navbar CSS 類別 |
| **logo** | string | 圖片選擇器 `[ImageEditor]` | — | Logo 圖片路徑 |
| **refreshPage** | bool | 核取方塊 `[CheckboxEditor]` | `false` | 切換時是否重新整理頁面 |
| **onRenderItem** | string | 事件編輯器 `[ScriptEditor]`（item） | — | 自訂渲染項目（預設回傳 `item.caption`） |
| **onLoadError** | string | 事件編輯器 `[ScriptEditor]`（err, url） | — | 載入錯誤事件 |

### Item 子項目屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **caption** | string | 文字 | — | 項目標題 |
| **alignment** | string | 選項 `[ItemsEditor]` | `"left"` | 對齊位置（`left` / `right`） |
| **subItems** | List\<SubItem\> | 集合編輯器 `[CollectionEditor]` | `[]` | 子選單項目 |
| **url** | string | 選單連結 `[MenuEditor]` | — | 連結目標（Panel） |
| **onClick** | string | 文字 | — | 點擊事件函式 |

### SubItem 子項目屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **caption** | string | 文字 | 子項目標題 |
| **url** | string | 選單連結 `[MenuEditor]` | 連結目標（Panel） |
| **onClick** | string | 文字 | 點擊事件函式 |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.menu`（`bootstrap.infolight.js` 第 18410–18525 行）

### 初始化流程

1. 解析 `data-options`，呼叫 `createMenu` 依 `items` 建立導覽列 `<li>` 項目。
2. 有 `subItems` 的項目建立為 Bootstrap Dropdown（`<li class="dropdown">`），並啟用 `dropdownHover()` 實現滑鼠懸停展開。
3. 項目依 `alignment` 屬性分配到 `.navbar-left` 或 `.navbar-right`。
4. 若 `refreshPage` 為 true，自動從 URL 參數 `m` 取得當前選單 ID 並標記為 active。

### 主要方法

| 方法 | 說明 |
|------|------|
| `createMenu()` | 根據 `items` 陣列建立選單 DOM。支援 `onRenderItem` 自訂項目 HTML。 |
| `getMenuItem(name)` | 依 `caption` 或 `url` 尋找選單項目，回傳 `<li>` DOM 元素。 |
| `active(menuItem)` | 啟動指定項目：移除舊 active、加入新 active。若項目有 `onClick` 則呼叫之；若有 `url` 且設定 `bindingObject`，依 `refreshPage` 決定重新導頁（修改 URL 的 `?m=` 參數）或透過 `$.loadHtml` 載入內容至綁定的 Panel。 |

### 頁面導航邏輯

- **`refreshPage = true`**：點擊項目後以 `window.location.href` 帶入 `?m=<url>` 參數重新整理頁面。若有 `#router_iframe` 則操作 `window.top`。
- **`refreshPage = false`**：透過 `$.loadHtml('#' + bindingObject, url)` 以 AJAX 動態載入內容至指定 Panel，不重整頁面。
- 點擊 `.navbar-brand`（Logo）時導回原始頁面（移除查詢字串）。

### 事件回呼

| 回呼 | 觸發時機 |
|------|----------|
| `onRenderItem(item)` | 渲染每個項目時呼叫，可自訂項目 HTML |
| `onLoadSuccess` | `$.loadHtml` 載入成功時觸發 |
| `onLoadError(err, url)` | `$.loadHtml` 載入失敗時觸發 |

## 備註

- MenuCls 為空時預設使用 `navbar-default`；可改為 `navbar-inverse` 等 Bootstrap 導覽列樣式。
- Logo 圖片高度固定為 30px（`style="height:30px"`）。
- 渲染結構：`<nav class="navbar">` → `<div class="container-fluid">` → navbar-header（含折疊按鈕、Logo）+ `<div class="bootstrap-menu">`（含 navbar-left、navbar-right）。
- 使用 `ItemDef["dataToggle"]` 取得 data-toggle 屬性名稱（相容不同 Bootstrap 版本）。
