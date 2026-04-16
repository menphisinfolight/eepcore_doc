# Card

> `EEPRWDTools.Core/Controls/Card.cs` — 55 行
> 繼承：`RWDContainerControl` → `RWDControl` → `Component`

## 用途

**卡片容器元件**（Card）。

Card 是一個可摺疊的容器面板，用於將子元件（childrens）包裝在卡片式 UI 中。支援標題、圖示、配色主題、顯示/隱藏控制，以及展開事件。當指定 `url` 時，子元件才會被渲染（延遲載入）。

## JSON 設定範例

```json
{
  "type": "card",
  "id": "cardOrder",
  "title": "訂單資訊",
  "iconCls": "glyphicon glyphicon-list-alt",
  "class": "blue",
  "commands": "moveTop,hide",
  "visible": true,
  "large": false,
  "horizontalColumnsCount": 200,
  "url": "/api/module",
  "childrens": []
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 `[Localization]` | — | 卡片標題（支援多語系） |
| **iconCls** | string | 圖示選擇器 `[IconRWDEditor("bootstrap")]` | — | Bootstrap 圖示 CSS 類別 |
| **class** | string | 下拉選擇 `[ItemsEditor]` | — | 配色主題（dark red / dark orange / dark blue / dark green / dark purple / dark yellow / dark gray / red / orange / blue / green / yellow / purple / gray），不傳入 data-options |
| **commands** | string | 多選 `[ItemsEditor]` | — | 卡片指令（moveTop / moveBottom / hide / open） |
| **url** | string | 文字 | — | 延遲載入 URL，有值時才渲染子元件 |
| **visible** | bool | 勾選 `[CheckboxEditor]` | `true` | 是否預設顯示 |
| **large** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否使用大尺寸模式 |
| **horizontalColumnsCount** | int | 數字框 `[NumberboxEditor]` | 200 | 水平欄數（不傳入 data-options） |
| **onOpen** | string | 腳本編輯器 `[ScriptEditor]` | — | 卡片展開時觸發，簽名 `function(){ return true; }` |

## 備註

- Card 繼承 `RWDContainerControl`，支援 `childrens` 子元件集合。
- Render 時產生 `<div class="info-card card-hidden {class}">` 標籤，CSS class 包含 `card-hidden` 表示初始隱藏，由前端 JS 控制顯示。
- `class` 和 `horizontalColumnsCount` 標記 `[DataOption(false)]`，不會輸出到 HTML `data-options` 屬性中。
