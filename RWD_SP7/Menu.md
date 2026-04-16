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

## 備註

- MenuCls 為空時預設使用 `navbar-default`；可改為 `navbar-inverse` 等 Bootstrap 導覽列樣式。
- Logo 圖片高度固定為 30px（`style="height:30px"`）。
- 渲染結構：`<nav class="navbar">` → `<div class="container-fluid">` → navbar-header（含折疊按鈕、Logo）+ `<div class="bootstrap-menu">`（含 navbar-left、navbar-right）。
- 使用 `ItemDef["dataToggle"]` 取得 data-toggle 屬性名稱（相容不同 Bootstrap 版本）。
