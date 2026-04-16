# Map (Editor)

> `EEPRWDTools.Core/Editors/Map.cs` — 25 行
> 繼承：`RWDEditor`

## 用途

**地圖編輯器**。在欄位中嵌入地圖顯示，支援設定地圖層級和值類型。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 200 | 地圖高度（px） |
| **valueType** | MapValueType | 列舉 | — | 值類型（座標格式） |
| **level** | int | 數字框 `[NumberboxEditor]` | 13 | 地圖縮放層級（1–13） |

## 備註

- 渲染為 `<div>` 加上 `bootstrap-geomap` CSS 類別。
