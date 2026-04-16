# Place (Editor)

> C# 端：`EEPRWDTools.Core/Editors/Place.cs` — 22 行
> 前端：`jquery.infolight.map.js` — `$.fn.place`（line 194–264，70 行）
> 繼承：`RWDEditor`

## 用途

**地點選取編輯器**。點擊地球按鈕後，透過瀏覽器定位 API 取得當前位置（經緯度或地址），自動填入欄位值。

C# 端只渲染 `<input class="bootstrap-place">`，前端 JS 包裝為 input + 地球圖示按鈕，點擊後呼叫 `$.fn.geomap.getCurrentPosition()` 取得裝置位置。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **valueType** | PlaceValueType | 列舉 | Address | 值類型：`Address`（地址文字）/ `Latlng`（經緯度） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀（停用按鈕） |

### ValueType 列舉

| 值 | 儲存格式 | 說明 |
|----|---------|------|
| `Address` | `台北市中山區南京東路一段` | 取得經緯度後反向地理編碼為地址文字 |
| `Latlng` | `25.0478,121.5319` | 直接儲存經緯度（小數點 4 位） |

## 前端行為（JavaScript）

> 原始碼：`jquery.infolight.map.js` line 194–264
> jQuery Plugin：`$.fn.place`

### HTML 渲染結構

```html
<div class="input-group">
  <input class="form-control bootstrap-place" disabled />
  <span class="input-group-btn">
    <button class="btn btn-default form-btn glyphicon glyphicon-globe" type="button"></button>
  </span>
</div>
```

輸入框預設 `disabled`，使用者無法手動輸入，只能透過按鈕定位。

### 公開 API 方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 回傳 `data('value')`（地址字串或經緯度） |
| `setValue(value)` | 設定 `data('value')` 並更新輸入框顯示 |
| `open()` | **核心方法**：觸發定位流程（見下方） |
| `readonly(value)` | `true` 停用地球按鈕、`false` 啟用 |

### 定位流程（open）

```
使用者點擊 🌐 按鈕
  → 顯示 loading
  → $.fn.geomap.getCurrentPosition(callback)
    → 成功取得位置：
      → valueType = 'latlng'：
          setValue("25.0478,121.5319")
      → valueType = 'address'：
          → $.fn.geomap.getAddress(lat, lng, callback)
          → 反向地理編碼取得地址
          → setValue("台北市中山區...")
    → 失敗：
      → 顯示 geolocationError 錯誤訊息
  → 隱藏 loading
```

### 定位服務（$.fn.geomap.getCurrentPosition）

依據系統設定的 MAP_TYPE 自動選擇地圖服務商：

| MAP_TYPE | 服務商 | 定位 API | 地址反查 API |
|----------|--------|---------|-------------|
| `baidu` | 百度地圖 | `BMap.Geolocation` | `BMap.Geocoder` |
| `qq` | 騰訊地圖 | `qq.maps.Geolocation` | `qq.maps.Geocoder` |
| `amap` | 高德地圖 | `AMap.Geolocation` | `AMap.Geocoder` |
| `google` | Google Maps | `google.maps.Geocoder` | `google.maps.Geocoder` |
| 其他（fallback） | 瀏覽器原生 | `navigator.geolocation.getCurrentPosition()` | 無 |

### 地圖服務設定

```javascript
// 在系統設定中定義（由伺服器端注入）
var MAP_TYPE = 'google';  // baidu / qq / amap / google
var MAP_KEY = 'your-api-key';
```

JS 動態載入對應的地圖 SDK：
```javascript
MAP_SERVICE = {
  'google': 'https://maps.googleapis.com/maps/api/js?key={0}',
  'baidu': 'https://api.map.baidu.com/api?v=2.0&ak={0}&s=1',
  'qq': 'https://map.qq.com/api/js?v=2.exp&key={0}',
  'amap': 'https://webapi.amap.com/maps?v=1.3&key={0}'
};
```

## ⚠️ HTTPS 必要條件

> **重要**：現代瀏覽器（Chrome 50+、Firefox 55+、Safari 10+）要求 **HTTPS** 才能使用 `navigator.geolocation.getCurrentPosition()`。HTTP 環境下定位 API 會被瀏覽器靜默拒絕（不會報錯，直接觸發 error callback）。

影響：
- **本機開發**（localhost）：定位可正常使用
- **HTTP 部署**：Place 元件的地球按鈕點擊後會顯示「定位失敗」
- **HTTPS 部署**：正常運作
- **百度/騰訊/高德 SDK**：部分服務商的 SDK 有自己的 HTTPS 要求

## 與 Map (Editor) 的差異

| 項目 | Place | Map (Geomap) |
|------|-------|-------------|
| **用途** | 取得位置值（填入表單欄位） | 顯示地圖（展示位置） |
| **互動** | 按鈕 → 定位 → 填值 | 顯示地圖標記 |
| **輸出** | 文字（地址或經緯度） | 地圖 `<div>` |
| **ValueType** | Address / Latlng | Latlng / Address / Current / CurrentAddress |
| **JS Plugin** | `$.fn.place` | `$.fn.geomap` |

## 備註

- 輸入框預設 `disabled`，使用者無法手動輸入地址，只能透過定位按鈕取得。
- 定位精度設定為 `enableHighAccuracy: true`，快取 30 分鐘（`maximumAge: 30min`）。
- 騰訊地圖額外載入 geolocation 工具庫（`apis.map.qq.com/tools/geolocation`）。
- fallback 使用瀏覽器原生 `navigator.geolocation`，不需要任何地圖 API Key，但無法反向地理編碼（只能取得經緯度，不能取得地址）。
