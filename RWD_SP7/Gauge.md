# Gauge

> `EEPRWDTools.Core/Controls/Gauge.cs` — 125 行
> 繼承：`RWDControl` → `Component`

## 用途

**儀表板元件**（Gauge）。

Gauge 以儀表板（儀錶盤）方式呈現單一數值指標。支援三種樣式：活動式（active）、甜甜圈式（donut）和區間式（zones）。可綁定資料來源動態取值，也可直接設定固定值。支援指針、色彩漸層、標籤、區間色帶等豐富的視覺自訂。前端渲染為 `<canvas class="bootstrap-gauge">`。

## JSON 設定範例

```json
{
  "type": "gauge",
  "id": "gaugeTemp",
  "remoteName": "qrySensor",
  "style": "zones",
  "field": "TEMPERATURE",
  "minField": "MIN_TEMP",
  "maxField": "MAX_TEMP",
  "currentValue": 30,
  "minValue": 0,
  "maxValue": 100,
  "size": 200,
  "angle": 5,
  "lineWidth": 2,
  "radius": 10,
  "viewLabels": true,
  "labelsSize": 20,
  "lablePosition": 0,
  "ptrLength": 5,
  "ptrStroke": 5,
  "ptrColor": "#000000",
  "colorStart": "#6FADCF",
  "colorStop": "#cccccc",
  "viewStaticLabels": false,
  "staticZones": [
    { "min": 0, "max": 60, "strokeStyle": "#00ff00" },
    { "min": 60, "max": 90, "strokeStyle": "#ff7f00" },
    { "min": 90, "max": 100, "strokeStyle": "#ff0000" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 資料來源名稱 |
| **style** | string | 下拉選單 `[ItemsEditor]` | "active" | 儀表板樣式（active/donut/zones） |
| **currentValue** | int | 數字框 `[NumberboxEditor]` | 30 | 目前值（設計時預覽用） |
| **minValue** | int | 數字框 `[NumberboxEditor]` | 0 | 最小值 |
| **maxValue** | int | 數字框 `[NumberboxEditor]` | 100 | 最大值 |
| **field** | string | 欄位選擇器 `[ColumnEditor]` | — | 數值欄位 |
| **minField** | string | 欄位選擇器 `[ColumnEditor]` | — | 最小值欄位（動態取值） |
| **maxField** | string | 欄位選擇器 `[ColumnEditor]` | — | 最大值欄位（動態取值） |
| **whereStr** | int | — | — | 查詢條件 |
| **size** | int | 數字框 `[NumberboxEditor]` | 200 | 儀表板尺寸（px） |
| **angle** | int | 數字框 `[NumberboxEditor]` | 5 | 儀表板角度 |
| **lineWidth** | int | 數字框 `[NumberboxEditor]` | 2 | 刻度線寬度 |
| **radius** | int | 數字框 `[NumberboxEditor]` | 10 | 儀表板半徑 |
| **viewLabels** | bool | 核取方塊 `[CheckboxEditor]` | true | 是否顯示刻度標籤 |
| **labelsSize** | int | 數字框 `[NumberboxEditor]` | 20 | 標籤文字大小 |
| **lablePosition** | int | 數字框 `[NumberboxEditor]` | 0 | 標籤位置偏移 |
| **ptrLength** | int | 數字框 `[NumberboxEditor]` | 5 | 指針長度 |
| **ptrStroke** | int | 數字框 `[NumberboxEditor]` | 5 | 指針寬度 |
| **ptrColor** | string | 顏色選擇器 `[ColorEditor]` | "#000000" | 指針顏色 |
| **colorStart** | string | 顏色選擇器 `[ColorEditor]` | "#6FADCF" | 漸層起始色 |
| **colorStop** | string | 顏色選擇器 `[ColorEditor]` | "#cccccc" | 漸層結束色 |
| **viewStaticLabels** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示靜態標籤 |
| **staticZones** | List\<ZoneRange\> | 集合編輯器 `[CollectionEditor]` | [] | 區間色帶定義（見下方子屬性） |

### ZoneRange 子屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **min** | int | 數字框 `[NumberboxEditor]` | — | 區間起始值 |
| **max** | int | 數字框 `[NumberboxEditor]` | — | 區間結束值 |
| **strokeStyle** | string | 顏色選擇器 `[ColorEditor]` | "#000000" | 區間顏色 |

## 備註

- Gauge 是唯一使用 `<canvas>` 標籤渲染的圖表元件，其他圖表皆使用 `<div>`。
- Gauge 的 Render 方法較特殊：會額外建立 `{id}_canvas` 容器 div 和 `{id}_text` 文字 div，結構為巢狀。
- `style` 三種模式：`active` 為基本活動儀表、`donut` 為環形儀表、`zones` 搭配 `staticZones` 顯示多色區間帶。
- `minField` 和 `maxField` 可讓最小/最大值從資料來源動態取得，不固定在設計時設定。
- 注意原始碼中 `lablePosition` 為拼寫錯誤（應為 labelPosition），但屬性名稱已定型不宜更改。
- ZoneRange 的屬性宣告為 `static`，這在程式設計上是一個潛在問題（所有 ZoneRange 實例共用同一值），但不影響 JSON 序列化時的行為。
- Gauge 沒有 onClick 或其他互動事件，僅為唯讀的數值展示元件。
