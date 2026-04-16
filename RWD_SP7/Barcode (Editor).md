# Barcode (Editor)

> `EEPRWDTools.Core/Editors/Barcode.cs` — 22 行
> 繼承：`RWDEditor`

## 用途

**條碼顯示編輯器**。將欄位值以條碼圖形顯示。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | 條碼高度（px） |
| **format** | BarcodeFormat | 列舉 | Code128 | 條碼格式：`Code39` / `Code128` / `Ean13` / `Ean128` |

## 前端行為（JavaScript）

> jQuery Plugin：`$.fn.BARcode`（`bootstrap.infolight.js` 第 12933–12975 行）

### 主要方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 回傳 `<input>` 的值。 |
| `setValue(value)` | 設定 `<input>` 值，移除舊的條碼 `<div>`，以 `$.fn.barcode(value, format, { barHeight })` 產生新條碼圖形。`format` 取自元件屬性（如 code128 等）。 |
| `readonly(value)` | `true` 時隱藏輸入框、顯示條碼圖形；`false` 時反之。 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-BARcode` CSS 類別。
