# RWDControl

> `EEPRWDTools.Core/Controls/RWDControl.cs` — 138 行

## 用途

**RWD 控制項基底類別**（Base Control）。

此檔案定義了 RWD 元件體系的三個核心抽象類別及兩個介面，是所有 RWD 控制項的繼承起點。

## 類別與介面一覽

### RWDControl（抽象類別）

繼承自 `Component`，實作 `IDataOption`。是所有 RWD 控制項的直接或間接基底類別。

| 成員 | 類型 | 說明 |
|------|------|------|
| `Render(HtmlString html)` | abstract method | 將控制項渲染為 HTML（子類別必須實作） |
| `DataOptionValues` | Dictionary\<string, object\> | `data-options` 屬性的鍵值對集合（IDataOption 實作） |
| `RenderTag(html, tag, tagClass, attrs)` | protected method | 產生帶有 `id` 和 `data-options` 的 HTML 標籤 |
| `GetToolItems()` | virtual method | 回傳工具列按鈕清單（預設空集合） |
| `GetControlTypes()` | virtual method | 回傳控制項類型名稱清單（預設為自身類別名） |
| `ItemDef` | static property | 取得 `RWDDef.Current.ItemDef`（Bootstrap 版本對應的 CSS class） |

### RWDContainerControl（抽象類別）

繼承自 `RWDControl`，是可包含子控制項的容器型控制項基底。

| 成員 | 類型 | 說明 |
|------|------|------|
| `Controls` | List\<RWDControl\> | 子控制項清單 |
| `RenderChildren(html)` | protected method | 依序呼叫所有子控制項的 `Render()` |
| `LoadProperties(JObject)` | override | 載入 JSON 屬性後，解析 `childrens` 陣列遞迴建立子控制項 |
| `GetComponent<T>(id)` | override | 遞迴搜尋子控制項（深度優先） |
| `GetComponents<T>()` | override | 遞迴收集所有指定型別的子控制項 |
| `GetControlTypes()` | override | 遞迴收集自身及所有子控制項的型別名稱 |

### RWDCollectionItem（抽象類別）

實作 `IDataOption`，用於非控制項但需要 `data-options` 的集合項目（如 Column 定義）。

### IFormColumn（介面）

繼承 `IDataOption`，定義表單欄位的共通屬性。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Title` | string | 欄位標題 |
| `Field` | string | 對應資料欄位名 |
| `NewRow` | bool | 是否換行 |
| `Span` | int | 佔用格數 |
| `Editor` | RWDEditor | 編輯器定義 |

### IToolItem（介面）

定義工具列按鈕項目。

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Text` | string | 按鈕文字 |
| `IconCls` | string | 圖示 CSS class |
| `Onclick` | string | 點擊事件 JS 函式 |

## 元件繼承結構

```
Component (EEPBase.Core)
  └─ RWDControl（所有控制項基底）
       ├─ RWDContainerControl（容器控制項基底）
       │    ├─ RWDPage
       │    ├─ Panel
       │    ├─ Tabs
       │    └─ ...
       ├─ DataPanel
       ├─ Literal
       └─ ...（各種控制項）
```

## 備註

- `LoadProperties` 中的子控制項 JSON 屬性名為 `childrens`（非標準英文複數，但為 EEP 慣用命名）。
- `GetComponent<T>` 使用深度優先搜尋，在巢狀結構中找到第一個符合的子控制項即回傳。
- `RenderTag` 是控制項渲染的基礎方法，自動將 `Id` 和 `DataOptionValues`（透過 `GetDataOptions()`）附加到 HTML 標籤上。
