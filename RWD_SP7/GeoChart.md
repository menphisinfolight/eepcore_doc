# GeoChart

> `EEPRWDTools.Core/Controls/GeoChart.cs` — 93 行
> 繼承：`RWDControl` → `Component`

## 用途

**地理圖表元件**（Geo Chart）。

GeoChart 載入 GeoJSON 地圖檔，將資料以色階（value）或氣泡（circle）方式在地圖區域上呈現。支援特殊區域設定（SpecialAreas）、地圖縮放/拖曳（Roam），以及 Circle 模式的半徑、透明度、顏色自訂。前端渲染為 `<div class="info-geochart">`。

## JSON 設定範例

```json
{
  "type": "geochart",
  "id": "gcTaiwan",
  "remoteName": "qryRegion",
  "jsonUrl": "taiwan.geojson",
  "chartType": "circle",
  "nameField": "CITY",
  "valueField": "POPULATION",
  "chartTitle": "各縣市人口",
  "subTitle": "2026年度",
  "roam": true,
  "width": 0,
  "height": 500,
  "maxRadius": 20,
  "circleOpacity": 0.7,
  "circleColor": "#ff6600",
  "areaColor": "#e7e8ea",
  "specialAreas": [
    { "areaName": "連江縣", "left": 120, "top": 30, "width": 2 }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **jsonUrl** | string | 檔案選擇器 `[DataFileEditor("geojson")]` | — | GeoJSON 地圖檔路徑 |
| **specialAreas** | List\<Area\> | 集合編輯器 `[CollectionEditor]` | [] | 特殊區域位置調整（見下方子屬性） |
| **chartType** | string | 下拉選單 `[ItemsEditor]` | "value" | 圖表類型（value = 色階 / circle = 氣泡） |
| **nameField** | string | 欄位選擇器 `[ColumnEditor]` | — | 地區名稱欄位 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 數值欄位 |
| **chartTitle** | string | — | — | 圖表主標題 |
| **subTitle** | string | — | — | 圖表副標題 |
| **roam** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否允許縮放/拖曳 |
| **width** | int | 數字框 `[NumberboxEditor]` | 0 | 寬度（0 = 自動） |
| **height** | int | 數字框 `[NumberboxEditor]` | — | 高度（px） |

### Circle 群組屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **maxRadius** | int | 數字框 `[NumberboxEditor]` | 15 | 氣泡最大半徑 |
| **circleOpacity** | double | 數字框 `[NumberboxEditor]`（小數） | 0.7 | 氣泡透明度（0~1） |
| **circleColor** | string | 顏色選擇器 `[ColorEditor]` | — | 氣泡顏色 |
| **areaColor** | string | 顏色選擇器 `[ColorEditor]` | "#e7e8ea" | 地圖區域底色 |

### 事件

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **onBeforeLoad** | string | 腳本編輯器 `[ScriptEditor]` | 載入前事件（參數：param） |
| **onLoad** | string | 腳本編輯器 `[ScriptEditor]` | 載入事件（參數：row） |
| **onClick** | string | 腳本編輯器 `[ScriptEditor]` | 點擊事件（參數：name, value） |

### Area 子屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **areaName** | string | — | — | 區域名稱 |
| **left** | int | 數字框 `[NumberboxEditor]` | 0 | 水平偏移 |
| **top** | int | 數字框 `[NumberboxEditor]` | 0 | 垂直偏移 |
| **width** | int | 數字框 `[NumberboxEditor]` | 1 | 區域縮放寬度 |

## 前端行為（JavaScript）

| 項目 | 說明 |
|------|------|
| **原始檔** | `jquery.infolight.chart.js` |
| **jQuery 外掛** | `$.fn.geochart`（透過 `$.createObj('geochart', ...)` 建立） |
| **CSS 選擇器** | `.info-geochart` |
| **圖表引擎** | ECharts（`echarts.init`、`echarts.registerMap`） |

### 方法

| 方法 | 說明 |
|------|------|
| `init` | 讀取 `data-options` 解析設定，設定寬高後呼叫 `loadJson` |
| `loadJson` | 以 `echarts.init` 建立圖表實例（Canvas 渲染器），透過 `$.get` 載入 GeoJSON 檔案，呼叫 `echarts.registerMap` 註冊地圖，再呼叫 `load` 載入資料。載入失敗時顯示錯誤訊息 |
| `load` | 組合查詢參數，透過 `$.loadData` 向後端取資料，呼叫 `onLoad` 回呼後呼叫 `loadData` |
| `loadData` | 依 `chartType` 分兩種模式渲染：**value** 模式使用 `type: 'map'` 以色階（lightskyblue → yellow → orangered）填色區域，搭配 `visualMap` 元件；**circle** 模式使用 `coordinateSystem: 'geo'` + `type: 'pie'` 在地圖上疊加氣泡圓餅，半徑依數值比例計算。繪製後綁定 ECharts click 事件，觸發 `onClick(name, value)` |
| `setWhere` | 更新 whereStr / whereItems 後重新呼叫 `load` |
| `resize` | 呼叫 ECharts 實例的 `chartObj.resize()` |

### 與其他圖表的差異

- GeoChart 不使用 jqPlot，而是使用 **ECharts** 引擎，因此初始化流程不同（需先載入 GeoJSON 並註冊地圖）。
- 不在 `bootstrap.infolight.js` 的 Form `getXxxChart` / `setWhere` 整合中（Form 僅整合 linechart、barchart、piechart、donutchart 四種）。
- 視窗 resize 事件（200ms debounce）會對所有 `.info-geochart` 呼叫 `resize`。

### Value 模式色階

色階漸層範圍：`lightskyblue` → `yellow` → `orangered`，由 ECharts 的 `visualMap` 元件自動依最小值到最大值映射。

## 備註

- GeoChart 使用 ECharts 地理圖表引擎，需搭配 GeoJSON 格式的地圖定義檔。
- `chartType` 為 `value` 時依數值以色階漸層填色區域；為 `circle` 時以氣泡大小表示數值。
- `specialAreas` 用於調整離島等區域的位置和大小，使其在地圖上更易辨識。
- GeoChart 的 onClick 事件參數為 `(name, value)`，與其他圖表的 `(ev, si, pi, data)` 不同。
- GeoChart 多了 `onLoad` 事件（參數：row），可在資料載入時逐筆處理。
