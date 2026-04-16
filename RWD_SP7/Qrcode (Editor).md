# Qrcode (Editor)

> `EEPRWDTools.Core/Editors/Qrcode.cs` — 19 行
> 繼承：`RWDEditor`

## 用途

**QR Code 顯示編輯器**。將欄位值以 QR Code 圖形顯示。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | QR Code 高度（px） |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-QRcode` CSS 類別。
