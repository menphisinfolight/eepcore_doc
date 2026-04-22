# Panel

> `EEPRWDTools.Core/Controls/Panel.cs` — 76 行
> 繼承：`RWDContainerControl` → `RWDControl` → `Component`

## 用途

**可收合面板容器元件**（Panel）。

Panel 是 RWD 工具箱的容器元件，用於將子元件包裹在一個具有標題列的面板中。當設定 `Title` 時，會產生 Bootstrap Panel 結構（含標題列與收合/展開按鈕）；未設定 `Title` 時則僅產生一個簡單的 `<div>` 包裹子元件。支援收合狀態、隱藏控制及背景色設定。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Title** | string | 文字 `[Localization("id")]` | — | 面板標題（支援多語系，以 id 為 key）。有值時產生可收合的 Panel 結構 |
| **Collapsed** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否預設收合。`true` 時面板內容區域隱藏，使用者可點擊展開 |
| **Hidden** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否隱藏整個面板。標記 `[Security("id")]` 可由權限控制 |
| **Background** | string | 色彩選擇器 `[ColorEditor]` | — | 面板背景色（CSS 色碼） |

## 技術參考（AI / 深度使用）

> 此區段彙整結構化內容 — JSON schema 範例、伺服器端渲染管線、渲染後 DOM 骨架 — 供 AI 檢索或需要深入理解元件行為的開發者參考。日常設定看前面的「設計介面屬性」即可。

### JSON 設定範例

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

### 渲染流程（伺服器端）

```
Panel.Render()
  ├─ Hidden == true          → 完全不輸出（整個面板省略）
  ├─ Title 有值（有標題列）  → 可收合面板結構
  │   <div.panel-group>
  │     <div.panel.panel-primary[ style="background:{Background}"]>
  │       <div.panel-heading>
  │         <h5> Title（多語系 key = id）</h5>
  │         <a data-toggle="collapse" href="#{id}_body">收合/展開按鈕</a>
  │       </div>
  │       <div.panel-collapse.[collapse | in] id="{id}_body">
  │         <div.panel-body>
  │           foreach children → children.Render()
  │         </div>
  │       </div>
  │     </div>
  │   </div>
  └─ Title 為空（單純容器）  → 只產出一個 <div> 包裹 children
      <div id="{id}" data-options="...">
        foreach children → children.Render()
      </div>
```

- `Collapsed == true` → `div.panel-collapse` 多一個 `collapse` class（初始摺疊）
- `Collapsed == false` → 改加 `in` class（初始展開）
- `Background` 走 inline style；原始碼裡的 `string.IsNullOrEmpty(Background)` 條件疑似寫反，實際行為要測過才準

### 渲染後 HTML 結構

**Title 有值**（典型用法）：

```html
<div class="panel-group" id="pnlBasic" data-options="...">
  <div class="panel panel-primary" style="background:#f5f5f5">
    <div class="panel-heading">
      <h5 class="panel-title">
        <a data-toggle="collapse" href="#pnlBasic_body">基本資料</a>
      </h5>
    </div>
    <div id="pnlBasic_body" class="panel-collapse in">
      <div class="panel-body">
        ...子元件渲染結果...
      </div>
    </div>
  </div>
</div>
```

**Title 為空**（純分組容器）：

```html
<div id="pnlBasic" data-options="...">
  ...子元件渲染結果...
</div>
```

`Collapsed` 與 `Hidden` 標記 `[DataOption(false)]`，不輸出到 `data-options`；僅 `Title`、`Background` 等輸出到前端。

## 備註

- Panel 類別標記了 `[DataOption]`，其屬性值會序列化為 `data-options` 屬性供前端使用。
- `Hidden` 屬性帶有 `[Security("id")]`，表示可由安全權限機制依面板 id 控制是否隱藏。
- `Collapsed` 和 `Hidden` 的 `[DataOption(false)]` 表示這兩個屬性不會輸出到 `data-options`，僅在伺服器端渲染時使用。
- 背景色的判斷邏輯中 `string.IsNullOrEmpty(Background)` 條件可能存在反向問題（應為 `!string.IsNullOrEmpty`），需留意實際行為。
