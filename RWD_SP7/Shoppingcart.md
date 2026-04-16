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

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 18610–18918 行
> jQuery 外掛名稱：`$.fn.shoppingcart`

### 公開 API 方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `options()` | — | 取得元件選項物件 |
| `init(options?)` | options | 初始化元件：以 Bootstrap Popover 建立購物車彈出面板。設定 `panelTitle`、`placement`、`trigger`，並套用自訂模板含 `popover-footer` |
| `bindEvent()` | — | 綁定面板內的互動事件：關閉按鈕、數量加減按鈕 |
| `load()` | — | 透過 `$.loadData` 載入購物車資料，完成後呼叫 `loadData` 渲染 |
| `refreshPopover()` | — | 重新渲染 Popover 內容：更新按鈕上的 badge 數量、產生商品清單 HTML（圖片、名稱、價格、數量加減）、呼叫 `onRenderFooter` 渲染頁尾 |
| `loadData(data)` | data: `{rows, total}` | 儲存資料至 `opts.datas` 並觸發 `refreshPopover` |
| `submit(callback)` | callback: Function | 將 `updatedRows` / `deletedRows` 透過 `$.updateData` 提交至後端，成功後呼叫 `acceptChanges` 並執行 callback |
| `add(row, callback?)` | row, callback | 新增商品至購物車。先載入最新資料，以 keys 比對是否已存在：存在則累加數量（updated），不存在則新增（inserted）。觸發 `onQuantityChanged` |
| `deleteAll(callback?)` | callback | 載入全部購物車資料後以 `$.updateData` 批次刪除所有項目 |
| `moveTo(remoteName, columnMatchs, callback?)` | remoteName, columnMatchs, callback | 將購物車內容搬移至指定 RemoteName：載入全部資料，依 `columnMatchs` 欄位對應（source→target）轉換後批次新增 |
| `acceptChanges()` | — | 清空暫存的 `insertedRows` / `updatedRows` / `deletedRows` |
| `close()` | — | 關閉 Popover 面板 |

### 關鍵前端行為

- **Popover 面板**：購物車以 Bootstrap Popover 實作，自訂模板包含 `popover-title`（含關閉按鈕）、`popover-content`（商品清單）、`popover-footer`（自訂頁尾）。
- **數量加減**：點擊 +/- 按鈕時立即更新 `row[quantityField]`，觸發 `onQuantityChanged` 後呼叫 `submit` 即時同步至後端。按鈕在請求期間設為 `disabled` 防止重複操作。
- **數量歸零自動刪除**：按 - 將數量減至 0 以下時，該商品自動轉為 `deletedRows` 並從畫面及資料中移除。
- **商品圖片**：圖片 URL 為 `../file?q=<imageField>&f=<imageFolder>` 格式，由後端檔案服務提供。
- **Badge 數量**：每次 `refreshPopover` 會更新觸發按鈕上的 `<span class="badge">` 顯示購物車總數。
- **priceFormatter**：透過 `opts.priceFormatter.call(target, row)` 自訂價格顯示（如幣別、千分位），未設定時價格區域為空。
- **onRenderFooter**：可自訂 Popover 底部 HTML（如小計、結帳按鈕），預設為空。

## 備註

- 渲染為 `<a class="btn bootstrap-shoppingcart">`，顯示 PanelTitle 文字，前端 JS 處理彈出面板邏輯。
- PriceFormatter 用於自訂價格顯示格式（如加上幣別符號、千分位）。
- OnRenderFooter 預設回傳空字串，可自訂購物車底部內容（如小計、總金額）。
