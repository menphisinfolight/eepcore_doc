# TABS

> `EEPRWDTools.Core/Controls/Tabs.cs` — 175 行
> 繼承：`RWDControl` → `Component`

## 用途

**頁籤容器元件**（Tabs）。

TABS 是 RWD 工具箱的頁籤容器，用於將多個子面板以頁籤（Tab）或膠囊（Pill）方式切換顯示。支援 Bootstrap 3 與 Bootstrap 5.3.3 兩種版本的渲染邏輯，設計師可設定頁籤模式、排列方式、欄位寬度等。每個頁籤（Tab）本身也是一個容器，可放置任意子元件。

## JSON 設定範例

```json
{
  "type": "tabs",
  "id": "mainTabs",
  "mode": "tab",
  "containerCls": "container-fluid",
  "tabCls": "col-xs-3 col-sm-12",
  "contentCls": "col-xs-9 col-sm-12",
  "justified": true,
  "stacked": false,
  "onSelect": "mainTabs_onSelect",
  "tabs": [
    {
      "id": "tab1",
      "title": "基本資料",
      "hidden": false,
      "childrens": []
    },
    {
      "id": "tab2",
      "title": "進階設定",
      "childrens": []
    }
  ]
}
```

## 設計介面屬性

### TABS 主元件

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Tabs** | `List<Tab>` | 集合編輯器 `[CollectionEditor]` | 空集合 | 頁籤集合，預設模板為 Tab1、Tab2 |
| **Mode** | string | 下拉選單 `[ItemsEditor]` | — | 頁籤模式：`tab`（標準頁籤）或 `pill`（膠囊樣式） |
| **ContainerCls** | string | 下拉選單 `[ItemsEditor]` | `""` | 外層容器 class：`container` 或 `container-fluid` |
| **TabCls** | string | 欄位寬度編輯器 `[ColumnClassEditor]` | `col-xs-3 col-sm-12` | 頁籤列的 Bootstrap 欄位 class |
| **ContentCls** | string | 欄位寬度編輯器 `[ColumnClassEditor]` | `col-xs-9 col-sm-12` | 內容區的 Bootstrap 欄位 class |
| **Justified** | bool | 勾選 `[CheckboxEditor(true)]` | `true` | 是否均分頁籤寬度（加入 `nav-justified` class） |
| **Stacked** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否垂直堆疊頁籤（加入 `nav-stacked` class） |
| **OnSelect** | string | 腳本編輯器 `[ScriptEditor]` | — | 頁籤切換事件。標記 `[DataOption]`，參數：`index`、`title` |

### Tab 內部類別

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Title** | string | 文字 `[Localization]` `[DataOption]` | — | 頁籤標題（支援多語系） |
| **Hidden** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否隱藏此頁籤。標記 `[Security("title")]` 可由權限控制 |

## 內部類別

### Tab

繼承 `RWDContainerControl`，代表單一頁籤頁面。

- **RenderHeader()**：渲染頁籤標題（`<li>` + `<a>`）。依 `RWDDef.RWD_Ver` 判斷版本：
  - Bootstrap 5.3.3：使用 `nav-item` / `nav-link` class 及 `data-bs-toggle`
  - Bootstrap 3：使用 `active` class 及 `data-toggle`
- **Render()**：渲染頁籤內容（`div.tab-pane`）。Bootstrap 5 使用 `fade show active`，Bootstrap 3 使用 `fade in active`。
- **Active**：公開欄位（非屬性），由 TABS 在渲染時自動設定第一個未隱藏的頁籤為 active。
- 每個 Tab 是一個 `RWDContainerControl`，內部可放置任意子元件（透過 `childrens` 載入）。

## 備註

- TABS 繼承 `RWDControl`（非 `RWDContainerControl`），因為子元件管理由 `Tab` 集合負責而非直接掛在 TABS 上。
- `GetControlTypes()` 和 `GetComponents<T>()` 會遞迴收集所有 Tab 內的子元件類型和元件實例。
- 若 Tab 未指定 `Id`，會自動以 `{TABS.Id}_{index}` 格式產生。
- `Hidden` 的頁籤在渲染時完全跳過（標題和內容都不輸出）。
- `OnSelect` 標記了 `[DataOption]`，會輸出到 `data-options` 供前端綁定頁籤切換事件。
