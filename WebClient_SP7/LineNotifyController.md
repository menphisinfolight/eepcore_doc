# LineNotifyController

> **檔案路徑**：`EEPWebClient.Core/Controllers/LineNotifyController.cs`（19 行）

## 用途

LINE Notify 回呼頁面，提供 LINE Notify OAuth 授權完成後的回呼端點，渲染對應的 View 頁面。

## URL 路由

預設 MVC 路由：`/LineNotify/{action}`

## Actions 表

| Action | HTTP | 路由 | 參數 | 回傳 | 說明 |
|--------|------|------|------|------|------|
| `Index` | GET | `/LineNotify` | 無 | ViewResult | 渲染 LINE Notify 回呼頁面 |

## 關鍵邏輯

- 使用 `[HttpGet, AddMessage, AddPublicKey, AddTheme]` 四個 Attribute
  - `AddMessage`：注入系統訊息至 ViewData
  - `AddPublicKey`：注入公鑰至 ViewData（用於前端加密）
  - `AddTheme`：注入佈景主題設定至 ViewData
- 實際邏輯僅回傳 `View()`，頁面內容由對應的 Razor View 處理

## 備註

- 屬於最簡單的控制器之一，僅提供頁面渲染
- LINE Notify 的實際通知發送邏輯不在此控制器，此處僅為 OAuth 回呼的 Landing Page
- 三個 Action Filter（`AddMessage`、`AddPublicKey`、`AddTheme`）為 EEP 系統共用的頁面初始化機制
