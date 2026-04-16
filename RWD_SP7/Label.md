# Label

> `EEPRWDTools.Core/Controls/Label.cs` — 76 行
> 繼承：`RWDControl` → `Component`

## 用途

**文字標籤元件**。以 `<p>` 標籤渲染文字內容，支援字型、顏色、背景、邊框、內距等樣式設定，以及 Bootstrap 文字樣式類別。文字中的換行符 `\n` 會轉為 `<br>`。

## JSON 設定範例

```json
{
  "type": "label",
  "id": "lblTitle",
  "text": "歡迎使用系統",
  "color": "#333333",
  "background": "#f5f5f5",
  "font": "bold 16px Arial",
  "labelCls": "text-center",
  "padding": "10px"
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **text** | string | 多行文字 `[TextareaEditor]` | `""` | 顯示文字（換行轉 `<br>`） |
| **color** | string | 色彩選擇器 `[ColorEditor]` | — | 文字顏色 |
| **background** | string | 色彩選擇器 `[ColorEditor]` | — | 背景色 |
| **padding** | string | 文字 | — | 內距（CSS padding） |
| **border** | string | 色彩選擇器 `[ColorEditor]` | — | 邊框顏色（`1px solid {色}`） |
| **font** | string | 字型選擇器 `[FontEditor]` | — | CSS font 設定（含底線偵測） |
| **labelCls** | string | 選項 `[ItemsEditor]` | — | Bootstrap 文字類別 |
| **onClick** | string | 事件編輯器 `[ScriptEditor]` | — | 點擊事件函式名稱 |

### LabelCls 可選值

`text-left`、`text-center`、`text-right`、`text-muted`、`text-primary`、`text-success`、`text-info`、`text-warning`、`text-danger`

## 備註

- Font 屬性若包含 `underline`，會自動拆分為 `text-decoration:underline` 並從 font 值中移除。
- OnClick 標記為 `[DataOption(false)]`，不輸出到 data-options，而是直接寫入 HTML 的 `onclick` 屬性。
