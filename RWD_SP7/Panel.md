# Panel

> `EEPRWDTools.Core/Controls/Panel.cs` — 76 行
> 繼承：`RWDContainerControl` → `RWDControl` → `Component`

## 用途

**可收合面板容器元件**（Panel）。

Panel 是 RWD 工具箱的容器元件，用於將子元件包裹在一個具有標題列的面板中。當設定 `Title` 時，會產生 Bootstrap Panel 結構（含標題列與收合/展開按鈕）；未設定 `Title` 時則僅產生一個簡單的 `<div>` 包裹子元件。支援收合狀態、隱藏控制及背景色設定。

## JSON 設定範例

```json
{
  "type": "panel",
  "id": "pnlBasic",
  "title": "基本資料",
  "collapsed": false,
  "hidden": false,
  "background": "#f5f5f5",
  "childrens": [
    { "type": "textbox", "id": "txtName" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Title** | string | 文字 `[Localization("id")]` | — | 面板標題（支援多語系，以 id 為 key）。有值時產生可收合的 Panel 結構 |
| **Collapsed** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否預設收合。`true` 時面板內容區域隱藏，使用者可點擊展開 |
| **Hidden** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否隱藏整個面板。標記 `[Security("id")]` 可由權限控制 |
| **Background** | string | 色彩選擇器 `[ColorEditor]` | — | 面板背景色（CSS 色碼） |

## 渲染邏輯

- 當 `Hidden = true` 時，整個面板不渲染。
- 當 `Title` 有值時：
  - 外層 `div.panel-group` → `div.panel.panel-primary`
  - 標題列 `div.panel-heading` 內含 `<h5>` 標題文字及收合按鈕（`<a>` 搭配 `data-toggle="collapse"`）
  - 內容區 `div.panel-collapse`（依 `Collapsed` 決定加上 `collapse` 或 `in` class）→ `div.panel-body` → 子元件
- 當 `Title` 為空時：僅產生一個帶 `id` 和 `data-options` 的 `<div>`，直接渲染子元件。

## 備註

- Panel 類別標記了 `[DataOption]`，其屬性值會序列化為 `data-options` 屬性供前端使用。
- `Hidden` 屬性帶有 `[Security("id")]`，表示可由安全權限機制依面板 id 控制是否隱藏。
- `Collapsed` 和 `Hidden` 的 `[DataOption(false)]` 表示這兩個屬性不會輸出到 `data-options`，僅在伺服器端渲染時使用。
- 背景色的判斷邏輯中 `string.IsNullOrEmpty(Background)` 條件可能存在反向問題（應為 `!string.IsNullOrEmpty`），需留意實際行為。
