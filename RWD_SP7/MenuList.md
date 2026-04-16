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

## 備註

- Render 輸出 `<div class="bootstrap-menulist">`，`font` 屬性會轉為 inline style。
- `font` 標記 `[DataOption(false)]`，不會輸出到 data-options；若包含 `underline`，會額外產生 `text-decoration:underline` 樣式。
- `onItemClick` 屬性名稱在程式碼中為小駝峰 `onItemClick`（注意大小寫）。
