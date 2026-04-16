# Qrcode (Editor)

> `EEPRWDTools.Core/Editors/Qrcode.cs` — 19 行
> 繼承：`RWDEditor`

## 用途

**QR Code 顯示編輯器**。將欄位值以 QR Code 圖形顯示。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | QR Code 高度（px） |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.QRcode`（`bootstrap.infolight.js` 第 12866–12931 行）

### 主要方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 回傳 `<input>` 的值。 |
| `setValue(value)` | 設定 `<input>` 值，移除舊的 QR Code `<div>`，以 `$.fn.qrcode` 產生新的 QR Code 圖形。圖形大小以 `height` 設定。內建 `utf16to8` 函式處理 Unicode 編碼轉換。 |
| `readonly(value)` | `true` 時隱藏輸入框、顯示 QR Code 圖形；`false` 時反之。 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-QRcode` CSS 類別。
