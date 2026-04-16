# MenuList

> `EEPRWDTools.Core/Controls/MenuList.cs` — 60 行
> 繼承：`RWDControl` → `Component`

## 用途

**選單列表元件**（Menu List）。

MenuList 以圖示方塊方式呈現系統選單項目，支援多種外觀樣式（方形、圓角、圓形、填滿），可自訂大小、配色、字型，以及項目渲染與點擊事件。

## JSON 設定範例

```json
{
  "type": "menulist",
  "id": "mlMain",
  "menuID": "MENU001",
  "width": 90,
  "height": 90,
  "style": "square",
  "margin": 3,
  "autoBackground": false
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **menuID** | string | 文字 | — | 選單 ID |
| **width** | int | 數字框 `[NumberboxEditor]` | 90 | 項目寬度（px） |
| **height** | int | 數字框 `[NumberboxEditor]` | 90 | 項目高度（px） |
| **background** | bool | 色彩選擇器 `[ColorEditor]` | — | 背景色 |
| **autoBackground** | bool | 勾選 `[CheckboxEditor]` | `false` | 是否自動配色 |
| **border** | bool | 色彩選擇器 `[ColorEditor]` | — | 邊框色 |
| **font** | string | 字型編輯器 `[FontEditor]` | — | 字型設定（不傳入 data-options） |
| **margin** | int | 數字框 `[NumberboxEditor]` | 3 | 項目間距（px） |
| **style** | string | 下拉選擇 `[ItemsEditor]` | `"square"` | 外觀樣式（square / roundSquare / circle / fill） |
| **onRenderItem** | string | 腳本編輯器 `[ScriptEditor]` | — | 項目渲染事件，簽名 `function(row){}` |
| **onItemClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 項目點擊事件，簽名 `function(row){}` |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.menulist`（`bootstrap.infolight.js` 第 18197–18408 行）

### 預設值

| 屬性 | 預設值 | 說明 |
|------|--------|------|
| `captionField` | `'text'` | 標題欄位路徑 |
| `iconField` | `'attributes.icon'` | 圖示欄位路徑（支援巢狀屬性） |
| `idField` | `'id'` | ID 欄位 |
| `childrenField` | `'children'` | 子節點欄位 |
| `padding` | 15 | 項目內距（px） |
| `iconRadio` | 0.3 | 圖示大小佔項目寬度比例 |
| `colors` | 20 色陣列 | 自動配色用的色票 |

### 初始化流程

1. 綁定 `click`（呼叫 `onItemClick` 或 `openMenu`）、`mouseenter`（加 box-shadow）、`mouseleave`（移除 box-shadow）事件。
2. 透過 AJAX POST `../main` 載入選單資料（`mode: 'getMenus'`；若 `menuID` 為 `'menufavor_'` 則用 `'getMenusFavor'`）。

### 主要方法

| 方法 | 說明 |
|------|------|
| `load()` | AJAX 載入選單資料後呼叫 `loadData`。 |
| `loadData(data)` | 清除舊內容，依 `menuID` 從樹狀資料中篩選對應子節點，逐一建立 `<li>` 元素。每個項目套用大小、間距、配色樣式。 |
| `openMenu(row)` | 呼叫 `window.top.getMenuUrl(row)` 取得 URL，再呼叫 `window.top.addTab()` 開啟新頁籤。 |

### 樣式處理

- **`circle`**：圖示區域設為圓形白字、彩色背景。
- **`fill`**：整個 `<li>` 填滿背景色，文字白色。
- **其他（square / roundSquare）**：圖示使用彩色文字、透明背景。
- `autoBackground` 為 true 時，依序從 20 色色票循環取色。
- `onRenderItem(row)` 回傳物件時套用為 CSS 樣式；回傳字串時加為 CSS class。

## 備註

- Render 輸出 `<div class="bootstrap-menulist">`，`font` 屬性會轉為 inline style。
- `font` 標記 `[DataOption(false)]`，不會輸出到 data-options；若包含 `underline`，會額外產生 `text-decoration:underline` 樣式。
- `onItemClick` 屬性名稱在程式碼中為小駝峰 `onItemClick`（注意大小寫）。
