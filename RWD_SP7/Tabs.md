# TABS

> `EEPRWDTools.Core/Controls/Tabs.cs` — 175 行
> 繼承：`RWDControl` → `Component`

## 用途

**頁籤容器元件**（Tabs）。

TABS 是 RWD 工具箱的頁籤容器，用於將多個子面板以頁籤（Tab）或膠囊（Pill）方式切換顯示。支援 Bootstrap 3 與 Bootstrap 5.3.3 兩種版本的渲染邏輯，設計師可設定頁籤模式、排列方式、欄位寬度等。每個頁籤（Tab）本身也是一個容器，可放置任意子元件。

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

## 前端行為（JavaScript）

Tabs **沒有自訂的 `$.createObj`**，直接使用 Bootstrap 原生的 Tab/Pill 元件。渲染後的 HTML 骨架見尾段「技術參考」。

### 前端控制範例

#### 切換到指定頁籤

```javascript
// 用 index（從 0 開始）
$('#mainTabs .nav-tabs a').eq(1).tab('show');  // 切到第 2 個頁籤

// 用 href
$('a[href="#mainTabs_1"]').tab('show');

// 切回第一個頁籤
$('#mainTabs .nav-tabs a:first').tab('show');
```

#### 隱藏/顯示頁籤

```javascript
// 隱藏第 2 個頁籤
$('a[href="#mainTabs_1"]').parent('li').hide();

// 顯示回來
$('a[href="#mainTabs_1"]').parent('li').show();

// 根據群組權限隱藏（常見用法）
function dfMaster_onLoad(row) {
    var groups = $.getVariableValue('groups');
    if (groups.indexOf("80") < 0) {
        $('a[href="#tabMaster_1"]').parent('li').hide();
    }
}
```

#### 停用頁籤（不可點擊）

```javascript
// 停用
$('#mainTabs .nav-tabs li').eq(2).addClass('disabled')
    .find('a').removeAttr('data-toggle');

// 重新啟用
$('#mainTabs .nav-tabs li').eq(2).removeClass('disabled')
    .find('a').attr('data-toggle', 'tab');
```

#### OnSelect 事件

```javascript
function mainTabs_onSelect(index, title) {
    // index 從 0 開始，title 為頁籤文字
    if (index === 1) {
        $('#dgDetail').datagrid('reload');  // 切到第 2 頁時重載 DataGrid
    }
}
```

#### 取得當前頁籤資訊

```javascript
var activeIndex = $('#mainTabs .nav-tabs li.active').index();   // 當前 index
var activeTitle = $('#mainTabs .nav-tabs li.active a').text();   // 當前 title
var tabCount = $('#mainTabs .nav-tabs li').length;               // 頁籤總數
```

#### 頁籤內整體 ReadOnly

```javascript
// 第 2 個頁籤內所有欄位設為 ReadOnly
$('#mainTabs_1').find('.form-control').setReadonly(true);
```

#### 動態新增頁籤

```javascript
var tabId = 'mainTabs_new';
$('#mainTabs .nav-tabs').append(
    '<li><a href="#' + tabId + '" data-toggle="tab">新頁籤</a></li>'
);
$('#mainTabs .tab-content').append(
    '<div class="tab-pane fade" id="' + tabId + '">新頁籤內容</div>'
);
```

### Tab 切換時的系統自動行為

`shown.bs.tab` 事件中自動處理（`bootstrap.infolight.js` line 370–410）：

1. **jSignature 重繪** — 頁籤隱藏時 canvas 尺寸歸零，切換回來時自動 resize
2. **notInitGrid 延遲載入** — 標記 `notInitGrid` 的頁籤，第一次切換到時才載入 DataGrid（節省初始載入時間）
3. **觸發 onSelect** — 取得 index 和 title 後呼叫設計師定義的 onSelect 回呼

## 技術參考（AI / 深度使用）

> 此區段彙整結構化內容 — JSON schema 範例、伺服器端渲染管線、渲染後 DOM 骨架 — 供 AI 檢索或需要深入理解元件行為的開發者參考。日常設定看前面的「設計介面屬性」、「前端控制範例」即可。

### JSON 設定範例

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

### 渲染流程（伺服器端）

```
Tabs.Render()
  ├─ 外層 container（ContainerCls）
  │   ├─ <ul.nav.nav-{mode}[.nav-justified][.nav-stacked]>  ← 頁籤列（TabCls）
  │   │   └─ foreach Tab（skip Hidden 的）
  │   │       └─ Tab.RenderHeader() → <li>/<li class="active"> + <a data-toggle="tab">
  │   └─ <div.tab-content>                                   ← 內容區（ContentCls）
  │       └─ foreach Tab（skip Hidden 的）
  │           └─ Tab.Render() → <div.tab-pane.fade[.in.active]>
  │               └─ foreach children → children.Render()
```

版本分支（由 `RWDDef.RWD_Ver` 決定）：
- **Bootstrap 3**：`data-toggle="tab"` + `.active` class + `.fade.in.active`
- **Bootstrap 5.3.3**：`data-bs-toggle="tab"` + `.nav-item` / `.nav-link.active` + `.fade.show.active`

第一個未隱藏（`!Hidden`）的 Tab 由 Tabs 渲染時自動設定 `Active = true`。Tab 未指定 `Id` 時自動以 `{TABS.Id}_{index}` 產生。

### 渲染後 HTML 結構

```html
<div class="bootstrap-tab" id="mainTabs" data-options="onSelect:mainTabs_onSelect">
  <ul class="nav nav-tabs nav-justified">
    <li class="active"><a href="#mainTabs_0" data-toggle="tab">基本資料</a></li>
    <li><a href="#mainTabs_1" data-toggle="tab">進階設定</a></li>
  </ul>
  <div class="tab-content">
    <div class="tab-pane fade in active" id="mainTabs_0">...子元件...</div>
    <div class="tab-pane fade" id="mainTabs_1">...子元件...</div>
  </div>
</div>
```

- `.bootstrap-tab`：Tabs 外層識別 class，`id` 對應設計介面的 TABS.Id
- `data-options`：僅 `OnSelect` 標記 `[DataOption]`；`Mode`/`Justified`/`Stacked` 等屬性會影響 class，不輸出到 `data-options`
- `<ul>` 的 class 組合：`nav nav-{mode}`（必）+ `nav-justified`（Justified）+ `nav-stacked`（Stacked）
- Bootstrap 5 版本：`<ul>` 與 `<li>` 改用 `nav-item` / `<a>` 改用 `nav-link`，`data-toggle` → `data-bs-toggle`

## 備註

- TABS 繼承 `RWDControl`（非 `RWDContainerControl`），因為子元件管理由 `Tab` 集合負責而非直接掛在 TABS 上。
- `GetControlTypes()` 和 `GetComponents<T>()` 會遞迴收集所有 Tab 內的子元件類型和元件實例。
- 若 Tab 未指定 `Id`，會自動以 `{TABS.Id}_{index}` 格式產生。
- `Hidden` 的頁籤在渲染時完全跳過（標題和內容都不輸出）。
- `OnSelect` 標記了 `[DataOption]`，會輸出到 `data-options` 供前端綁定頁籤切換事件。
- Tabs 使用 Bootstrap 原生 Tab API（`$().tab('show')`），不是 EEP 自訂的 jQuery Plugin。
