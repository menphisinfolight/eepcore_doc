# Security

> `EEPRWDTools.Core/Controls/Security.cs` — 18 行
> 繼承：`RWDControl` → `Component`

## 用途

**安全性元件**（Security）。

Security 為標記型元件，不產生任何 HTML 輸出（Render 方法為空）。用於在模組中宣告安全性相關的 data-options 設定，由前端框架讀取處理。

## 備註

- 標記 `[DataOption]`，但無任何自訂屬性。
- Render 方法為空實作，不輸出任何 HTML。
