# Pivottable

> `EEPRWDTools.Core/Controls/Pivottable.cs` — 82 行
> 繼承：`RWDControl` → `Component`

## 用途

**樞紐分析表元件**（Pivot Table）。

Pivottable 提供互動式樞紐分析功能，可從資料來源載入資料後，由使用者拖放欄位進行多維度交叉分析。支援自訂聚合函數、渲染器，以及預設的列/欄配置。

## JSON 設定範例

```json
{
  "type": "pivottable",
  "id": "pvSales",
  "remoteName": "cmdSalesData",
  "panelHeight": 200,
  "digitsAfterDecimal": 2,
  "defaultColumn": "AMOUNT",
  "columns": [
    { "title": "產品", "field": "PRODUCT", "tableName": "" }
  ],
  "rows": [
    { "title": "區域", "field": "REGION", "tableName": "" }
  ],
  "showColumns": [
    { "title": "金額", "field": "AMOUNT", "tableName": "", "hideInValue": false }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源命令 |
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 欄（Column）區域的欄位集合 |
| **rows** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | — | 列（Row）區域的欄位集合 |
| **showColumns** | List\<ShowColumn\> | 集合編輯器 `[CollectionEditor]` | — | 顯示欄位集合 |
| **defaultColumn** | string | 欄位選擇器 `[ColumnEditor]` | — | 預設值欄位 |
| **whereStr** | string | — | — | 查詢條件字串 |
| **panelHeight** | int | 數字框 `[NumberboxEditor]` | 200 | 面板高度（px） |
| **digitsAfterDecimal** | int | 數字框 `[NumberboxEditor]` | 2 | 小數位數 |
| **aggregators** | string | 樞紐聚合器 `[PivotAggregatorsEditor]` | — | 聚合函數設定 |
| **renderers** | string | 樞紐渲染器 `[PivotRendersEditor]` | — | 渲染器設定 |
| **queryObj** | string | — | — | 查詢物件 |
| **onLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 資料載入後事件，簽名 `function(data){}` |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 儲存格點擊事件，簽名 `function(item){}` |

### Column 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題 |
| **field** | string | 欄位名稱 |
| **tableName** | string | 資料表名稱 |

### ShowColumn 子項目

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題 |
| **field** | string | 欄位名稱 |
| **tableName** | string | 資料表名稱 |
| **hideInValue** | bool | 是否在值區域中隱藏 |

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 16001–16974 行
> jQuery 插件名稱：`$.fn.pivottable`（透過 `$.createObj('pivottable', ...)` 建立）

### HTML 結構

```
div.bootstrap-pivottable        ← 原始元件 div
  .pvtUi                        ← pivotUI 自動產生的互動式介面
    .pvtUnused                  ← 未使用欄位區域（拖放來源）+ 匯出按鈕
    .pvtVals                    ← 值區域（聚合函數 + 值欄位下拉選單）
    .pvtAxisContainer.pvtHorizList  ← 欄（Column）軸拖放區
    .pvtAxisContainer.pvtRows      ← 列（Row）軸拖放區
    .pvtRendererArea            ← 渲染結果區域
      table.pvtTable            ← 樞紐表格（表格類渲染器）
        th.pvtAxisLabel         ← 軸標籤
        th.pvtColLabel          ← 欄標籤
        th.pvtRowLabel          ← 列標籤
        td.pvtVal               ← 資料儲存格（含 data-value 屬性）
        td.pvtTotal             ← 合計儲存格
        td.pvtGrandTotal        ← 總計儲存格
    .pvtAttrDropdown            ← 各欄位的值篩選下拉選單
```

### 公開 API 方法

| 方法 | 簽名 | 說明 |
|------|------|------|
| `init` | `(jq, options)` | 初始化：若設定 `alwaysClose` 則預設 whereStr 為 `1=0`；若指定 `renderObjectID` 則將元件移入目標容器；自動呼叫 `load` |
| `options` | `(jq)` → object | 取得元件設定物件 |
| `load` | `(jq)` | 觸發資料載入：先呼叫 `onBeforeLoad` 回呼，再呼叫 `loadData` |
| `loadData` | `(jq)` | 透過 `$.loadData` 向後端查詢資料，成功後存入 `data('pivottable').data`，呼叫 `renderDIV` 渲染，觸發 `onLoad` 回呼，並附加匯出按鈕 |
| `getData` | `(jq)` → array | 取得已載入的原始資料陣列 |
| `renderDIV` | `(jq, options)` | 核心渲染方法：將原始資料轉換為以 title 為 key 的顯示資料，設定 renderers/aggregators/rows/cols 後呼叫 `pivotUI` |
| `exportExcel` | `(jq)` | 將 `.pvtTable` 的 HTML 表格結構（含 colspan/rowspan）序列化為 JSON，POST 至 `../main/file` 匯出 Excel |
| `setWhere` | `(jq, where)` | 設定查詢條件（支援陣列或字串），設定後自動重新 `loadData` |
| `getWhereItem` | `(jq, op?)` | 將 `whereItems` 陣列組合為 SQL where 字串（支援 `=`、`%`、`%%` 條件） |
| `exportData` | `(jq, options)` | 匯出資料（目前程式碼已註解，保留介面） |

### 聚合函數對照表

`aggregators` 設定值為以分號分隔的索引，對應的聚合函數如下（第 16194–16220 行）：

| 索引 | 函數 | 說明 |
|------|------|------|
| 0 | `sum` | 加總 |
| 1 | `count` | 計數 |
| 2 | `countUnique` | 不重複計數 |
| 3 | `listUnique` | 列出不重複值 |
| 4 | `sum (Integer)` | 整數加總 |
| 5 | `average` | 平均 |
| 6 | `min` | 最小值 |
| 7 | `max` | 最大值 |
| 8 | `sumOverSum` | 比率（Sum/Sum） |
| 9 | `sumOverSumBound80 (upper)` | 80% 上界 |
| 10 | `sumOverSumBound80 (lower)` | 80% 下界 |
| 11 | `fractionOf count/total` | 計數佔總計百分比 |
| 12 | `fractionOf count/row` | 計數佔列百分比 |
| 13 | `fractionOf count/col` | 計數佔欄百分比 |
| 14 | `fractionOf sum/total` | 加總佔總計百分比 |
| 15 | `fractionOf sum/row` | 加總佔列百分比 |
| 16 | `fractionOf sum/col` | 加總佔欄百分比 |

### 渲染器對照表

`renderers` 設定值為以分號分隔的索引，對應的渲染器如下（第 16440–16480 行）：

| 索引 | 渲染器 | 說明 |
|------|--------|------|
| 0 | Table | 標準樞紐表格 |
| 1 | Table Barchart | 表格內嵌橫條圖 |
| 2 | Heatmap | 熱力圖（全域） |
| 3 | Row Heatmap | 熱力圖（列方向） |
| 4 | Col Heatmap | 熱力圖（欄方向） |
| 5 | Line Chart (C3) | 折線圖 |
| 6 | Bar Chart (C3) | 長條圖 |
| 7 | Stacked Bar Chart (C3) | 堆疊長條圖 |
| 8 | Area Chart (C3) | 面積圖 |
| 9 | Scatter Chart (C3) | 散佈圖 |

### 關鍵行為

1. **資料轉換與渲染**（`renderDIV`，第 16106–16897 行）
   - 從 `showColumns` 取得 `field` → `title` 對應，將後端 row 資料轉為以 title 為 key 的物件陣列（`showData`）。
   - 若無資料，仍推入一筆空 row 以確保 pivotUI 能正確顯示欄位。
   - 從 `rows` 與 `columns` 設定取出 title 作為 pivotUI 的 `rows` / `cols` 參數。
   - `defaultColumn` 對應至 `vals` 參數（預設值欄位）。

2. **hideInValue 欄位隱藏**（第 16502–16520 行、第 16877–16895 行）
   - `showColumns` 中設定 `hideInValue=true` 的欄位，會在渲染完成後透過 `setTimeout` 找到 `.pvtAttrDropdown` 中對應選項並設為 `display:none`。

3. **儲存格點擊事件**（`pivottableRenderer` 內，第 16633–16681 行）
   - 每個 `td.pvtVal` 綁定 `onclick`，從 CSS class 解析 row/col 索引。
   - 組合出 `whereItems`（含 row 軸欄位值 + col 軸欄位值），傳入 `options.onClick` 回呼。
   - 回呼參數包含 `value`（儲存格值）、`columns`（欄軸篩選條件）、`rows`（列軸篩選條件）、`whereItems`（合併條件陣列）。

4. **C3 圖表整合**（`makeC3Chart`，第 16221–16439 行）
   - 支援 line / bar / stacked bar / area / scatter 五種圖表類型。
   - 圖表尺寸預設為視窗寬高的 1/1.4。
   - scatter 類型使用 `xs` 資料格式；其他類型使用 category X 軸。
   - 提供 20 色的色盤（Google Charts 配色）。
   - 圖表先在隱藏的 `renderArea` div 中以 `c3.generate` 渲染，完成後 detach 回傳。

5. **自訂數值格式**（`numberFormat`，第 16160–16183 行）
   - 小數位數由 `options.digitsAfterDecimal` 控制。
   - 支援千分位分隔符、前後綴、零值隱藏。
   - 衍生出 `usFmt`（標準）、`usFmtInt`（整數）、`usFmtPct`（百分比）三種格式器。

6. **匯出 Excel**（`exportExcel`，第 16064–16097 行）
   - 遍歷 `.pvtTable` 所有 `tr` / `td` / `th`，收集 `colspan`、`rowspan`、`innerHTML`、`data-value`。
   - 以 JSON 格式 POST 至 `../main/file?mode=exportTable`，伺服器回傳檔案路徑後觸發下載。

### 依賴套件

- `PivotTable.js`（`$.fn.pivotUI`、`$.pivotUtilities`）— 樞紐分析核心套件
- `C3.js`（`c3.generate`）— 圖表渲染引擎

## 備註

- Render 輸出 `<div class="bootstrap-pivottable">`，前端使用 PivotTable.js 套件。
- `columns` 和 `rows` 使用相同的 `Column` 子類別定義。
- `aggregators` 和 `renderers` 使用專屬的設計介面編輯器（`PivotAggregatorsEditor` / `PivotRendersEditor`）。
