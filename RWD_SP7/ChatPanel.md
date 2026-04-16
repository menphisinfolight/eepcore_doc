# ChatPanel

> `EEPRWDTools.Core/Controls/ChatPanel.cs` — 98 行
> 繼承：`RWDControl` → `Component`

## 用途

**聊天面板元件**（Chat Panel）。

ChatPanel 提供即時對話介面，可串接後端處理器（Processor）實現 AI 對話或客服功能。支援歷史訊息保留、訊息壓縮率設定，以及傳送/接收事件攔截。

## JSON 設定範例

```json
{
  "type": "chatpanel",
  "id": "cpAssistant",
  "title": "AI 助手",
  "helloWorld": "您好，有什麼可以幫您？",
  "userIconCls": "glyphicon glyphicon-user",
  "gptIconCls": "glyphicon glyphicon-cloud",
  "placeHolder": "請輸入訊息...",
  "proc": "chatProcessor",
  "keepHistory": true,
  "height": 400,
  "compressRate": 0
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **id** | string | 文字 | — | 元件識別碼 |
| **title** | string | 文字 | — | 面板標題 |
| **helloWorld** | string | 文字 | — | 初始歡迎訊息 |
| **userIconCls** | string | 圖示選擇器 `[IconRWDEditor("bootstrap")]` | — | 使用者訊息圖示 |
| **gptIconCls** | string | 圖示選擇器 `[IconRWDEditor("bootstrap")]` | — | AI/機器人訊息圖示 |
| **placeHolder** | string | 文字 | — | 輸入框提示文字 |
| **proc** | string | 選單編輯器 `[MenuEditor("processor")]` | — | 後端處理器名稱 |
| **keepHistory** | bool | — | — | 是否保留對話歷史 |
| **height** | int | 數字框 `[NumberboxEditor]` | — | 聊天區域高度（px，範圍 100–400） |
| **compressRate** | double | 數字框 `[NumberboxEditor]` | — | 訊息壓縮率（0–1，小數 2 位） |
| **onLoad** | string | 腳本編輯器 `[ScriptEditor]` | — | 載入事件，簽名 `function(){}` |
| **onSend** | string | 腳本編輯器 `[ScriptEditor]` | — | 傳送訊息事件，簽名 `function(message){ return message; }` |
| **onReceive** | string | 腳本編輯器 `[ScriptEditor]` | — | 接收訊息事件，簽名 `function(message){ return message; }` |

## 備註

- Render 輸出完整的 Bootstrap Panel 結構：標題列（可摺疊）、聊天訊息區域、底部輸入區（含文字輸入框、檔案上傳按鈕、Send 按鈕）。
- 聊天區域高度由 `height` 屬性控制，設為 inline style `height:{Height}px;overflow-y:auto`。
- 面板支援摺疊功能，透過 Bootstrap `data-toggle="collapse"` 實現。
- `onSend` 和 `onReceive` 可攔截並修改訊息內容，回傳修改後的訊息字串。
