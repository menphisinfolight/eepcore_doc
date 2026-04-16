# Barcode (Editor)

> `EEPRWDTools.Core/Editors/Barcode.cs` — 22 行
> 繼承：`RWDEditor`

## 用途

**條碼顯示編輯器**。將欄位值以條碼圖形顯示。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | 條碼高度（px） |
| **format** | BarcodeFormat | 列舉 | — | 條碼格式 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-BARcode` CSS 類別。
