# Breadcrumb

> `EEPRWDTools.Core/Controls/Breadcrumb.cs` — 52 行
> 繼承：`RWDControl` → `Component`

## 用途

**麵包屑導覽元件**。以 Bootstrap Breadcrumb 樣式渲染階層導覽路徑，支援對齊方式、前置圖示、多個導覽項目。每個項目可設定連結目標或綁定 Panel。

## JSON 設定範例

```json
{
  "type": "breadcrumb",
  "id": "bcNav",
  "alignment": "left",
  "iconCls": "glyphicon-home",
  "items": [
    { "caption": "首頁", "url": "home" },
    { "caption": "產品列表", "bindingObject": "pnlProducts" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **alignment** | string | 選項 `[ItemsEditor]` | `"right"` | 對齊方式 |
| **iconCls** | string | 圖示選擇器 `[IconRWDEditor]` | — | 前置 Glyphicon 圖示 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | `[]` | 導覽項目集合（預設含「首頁」） |

### Alignment 可選值

`left`、`center`、`right`

### Item 子項目屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **caption** | string | 文字 | 項目標題 |
| **url** | string | 選單連結 `[MenuEditor]` | 連結目標（Panel） |
| **bindingObject** | string | 元件選擇器 `[ControlEditor]`（panel） | 綁定的 Panel 元件 |

## 備註

- 項目間分隔符號為 ` > `，透過內嵌 `<style>` 定義 `.breadcrumb > li + li:before` 樣式。
- 若設定 `iconCls`，會在麵包屑列表前顯示 Glyphicon 圖示。
- CollectionEditor 預設建立值為 `["首頁", ""]`（一個「首頁」項目）。
