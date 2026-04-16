# Shoppingcart

> `EEPRWDTools.Core/Controls/Shoppingcart.cs` — 54 行
> 繼承：`RWDControl` → `Component`

## 用途

**購物車元件**。以彈出面板方式顯示購物車內容，包含商品圖片、名稱、數量等資訊。支援數量變更事件和自訂頁尾格式化（如顯示總金額）。

## JSON 設定範例

```json
{
  "type": "shoppingcart",
  "id": "cart",
  "remoteName": "cmdCartItems",
  "nameField": "PRODUCT_NAME",
  "imageField": "PRODUCT_IMAGE",
  "quantityField": "QTY",
  "panelTitle": "購物車",
  "placement": "bottom",
  "pageSize": 400
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源 |
| **nameField** | string | 欄位選擇器 `[ColumnEditor]` | — | 商品名稱欄位 |
| **imageFolder** | string | 文字 | — | 圖片資料夾路徑 |
| **imageField** | string | 欄位選擇器 `[ColumnEditor]` | — | 商品圖片欄位 |
| **priceFormatter** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 價格格式化函式 |
| **quantityField** | string | 欄位選擇器 `[ColumnEditor]` | — | 數量欄位名稱 |
| **panelTitle** | string | 文字 | `"Shopping Cart"` | 購物車面板標題 |
| **pageSize** | int | 數字框 `[NumberboxEditor]`（min:100, max:400） | `400` | 每頁顯示筆數上限 |
| **placement** | string | 選項 `[ItemsEditor]` | `"auto"` | 彈出面板位置 |
| **onRenderFooter** | string | 事件編輯器 `[ScriptEditor]`（datas） | — | 頁尾格式化函式（回傳 HTML） |
| **onQuantityChanged** | string | 事件編輯器 `[ScriptEditor]`（row） | — | 數量變更事件 |

### Placement 可選值

`top`、`bottom`、`left`、`right`（設計介面預設 `auto`）

## 備註

- 渲染為 `<a class="btn bootstrap-shoppingcart">`，顯示 PanelTitle 文字，前端 JS 處理彈出面板邏輯。
- PriceFormatter 用於自訂價格顯示格式（如加上幣別符號、千分位）。
- OnRenderFooter 預設回傳空字串，可自訂購物車底部內容（如小計、總金額）。
