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

## 前端行為（JavaScript）

> 來源：`bootstrap.infolight.js` 第 20464–20803 行（約 340 行）

### 元件註冊

以 `$.fn.chatpanel = $.createObj('chatpanel', {...})` 註冊為 jQuery 插件。

### init 初始化流程

1. 解析 options（從 HTML `data-options` 或程式傳入）。
2. 觸發 `onLoad` 回呼。
3. 設定 Send 按鈕文字為本地語系（`$.fn.locale.send`）。
4. 設定輸入框 placeholder（若有 `placeHolder`）。
5. 若有 `helloWorld`，以 `from: 'example'` 類型附加歡迎訊息（HTML 格式）。
6. 建立圖片放大 modal（`#imgZoomModal`），支援拖曳（draggable）。
7. 呼叫 `createToolItem` 建立工具列。
8. 呼叫 `bindEvent` 綁定事件。

### 可設定屬性（options，JS 層額外識別的）

| 屬性 | 類型 | 說明 |
|------|------|------|
| **proc** | string | 後端處理器名稱，透過 `$.callProc` 呼叫 |
| **placeHolder** | string | 輸入框 placeholder 文字 |
| **helloWorld** | string | 初始歡迎訊息（以 HTML 渲染） |
| **keepHistory** | bool | `getLastText` 時是否包含所有歷史訊息（true = 全部；false = 僅最後一則 AI 回覆） |
| **endingWord** | string | 結束標記詞，AI 回覆含此詞時自動觸發清除流程 |
| **userIconCls** | string | 使用者訊息圖示 class（預設 `fa-user`） |
| **gptIconCls** | string | AI 訊息圖示 class（預設 `fa-android`） |
| **typeItems** | Array\<{value, text}\> | 下拉選單選項（呈現為 `<select>` 在聊天面板內） |
| **onLoad** | function | 載入/清除時觸發 |
| **onSend** | function | 傳送訊息前觸發，`function(text)`，回傳 `false` 取消傳送，回傳字串替換訊息內容 |
| **onReceive** | function | 接收回覆後觸發，`function(result)`，回傳 `false` 不顯示，回傳字串替換內容 |

### 公開方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `init` | options? | 初始化元件 |
| `chatPanel` | — | 回傳 `.chat-panel` 的 jQuery 物件 |
| `createToolItem` | — | 建立清除按鈕及 typeItems 下拉選單 |
| `bindEvent` | — | 綁定 Send / Enter / 檔案上傳 / 清除事件 |
| `sendChat` | — | 傳送訊息（詳見下方流程） |
| `appendChat` | chat 物件 | 新增一則訊息至聊天面板 |
| `processing` | — | 進入「思考中」狀態 |
| `processed` | — | 結束「思考中」狀態 |
| `clear` | — | 清空對話區域，重建工具列，重新顯示 helloWorld |
| `getLastText` | — | 取得最後（或全部）對話內容文字 |
| `setValue` | value, imageContent? | 外部設值，支援圖片（`img:` 前綴的 base64） |

### createToolItem 方法

在 `.chat-panel` 底部建立工具列：
- **清除按鈕**（`btn-primary chat-clear`）：文字為 `$.fn.locale.clear`。
- **類型下拉選單**（僅在 `typeItems` 有值時）：`<select>` 元素，寬 120px，值隨 `sendChat` 一起傳送。

### bindEvent 事件綁定

- **檔案上傳按鈕**（`.file-submit`）：設定 `data-loading-text` 為上傳中文字，點擊呼叫 `$.submitChatFile(target)`。
- **Send 按鈕**（`.chat-send`）：呼叫 `sendChat`。
- **清除按鈕**（`.chat-clear`）：呼叫 `clear`。
- **輸入框 Enter 鍵**：按 Enter 觸發 `sendChat` 並阻止預設行為（不換行）。
- **輸入框 blur**：若 `isEnd` 標記存在，自動清除對話（配合 `endingWord` 機制）。

### sendChat 訊息傳送流程

1. 取得輸入框文字。
2. 觸發 `onSend` 回呼：回傳 `false` 取消；回傳字串替換文字。
3. 移除 `.example` 元素（歡迎訊息）。
4. 以 `getLastText` 取得先前對話脈絡。
5. 以 `appendChat` 將使用者訊息加入面板（`from: 'user'`，含動畫）。
6. 進入 `processing` 狀態。
7. 呼叫 `$.callProc(opts.proc, { chat, image, last, type })`：
   - `chat`：使用者輸入文字
   - `image`：圖片 base64 資料（如有）
   - `last`：先前對話脈絡
   - `type`：typeItems 下拉選單的值
8. **成功回呼**：
   - 呼叫 `processed` 結束思考狀態。
   - 觸發 `onReceive` 回呼（可攔截/修改）。
   - 檢查 `endingWord`：若回覆含結束詞，移除結束詞、設 `isEnd` 標記、聚焦輸入框（blur 時自動清除）。
   - 以 `appendChat` 將 AI 回覆加入面板（`from: 'ai'`，含動畫）。
9. **失敗回呼**：以 `from: 'error'` 顯示錯誤訊息。

### appendChat 訊息渲染

接收 chat 物件 `{ from, value, animate, isHtml, after, t }`：

- **from 類型**：`'user'`、`'ai'`、`'example'`、`'error'`（決定 CSS class 與圖示）。
- **圖示**：
  - `user` → `opts.userIconCls`（預設 `fa-user`）
  - `ai` → `opts.gptIconCls`（預設 `fa-android`）
  - `example` / `error` → 無圖示
- **圖片處理**：若 `value` 包含 `img:` 前綴，將 base64 渲染為 `<img>` 元素（高度 128px），點擊可在 modal 中放大檢視。
- **文字處理**：
  - `isHtml: true` → 直接渲染 HTML
  - 否則轉義 `<` `>` 並將 `\n` 轉為 `<br>`
- **動畫**：`animate: true` 時，先隱藏訊息，捲動至底部後以 `fadeIn(300)` 淡入。
- **定位**：有 `after` 時插入至指定元素之後，否則附加至面板底部。
- 訊息資料儲存於 DOM 元素的 `data('value')` 中。

### processing / processed 思考狀態

- **processing**：
  - 禁用輸入框、清空內容。
  - 顯示 AI 圖示 + 「正在思考」文字，以 600ms 間隔切換 `.` / `..` / `...` 動畫。
- **processed**：
  - 清除 interval、啟用輸入框、移除 `.processing` 元素。

### getLastText 對話脈絡

- `keepHistory: true` → 收集所有 `div.ai` 和 `div.user` 的訊息，格式為 `from:value`，以 `\r\n` 連接。
- `keepHistory: false` → 僅取最後一則 `div.ai` 訊息。
- 訊息值從 `.chat-message` 或 `.imgzoom` 元素的 `data('value')` 中取得。

### setValue 方法

- 將值拆分（`;` 或 `,`），存入元素 data。
- 若值含 `img:` 前綴，儲存為 `image` data 並直接以 `appendChat` 顯示（`from: 'user'`）。
- 否則清除相關的 link 和 close 按鈕。

### clear 方法

1. 觸發 `onLoad` 回呼（重設狀態）。
2. 移除 `.chat-panel` 內所有 `<div>` 子元素。
3. 重建工具列（`createToolItem`）。
4. 若有 `helloWorld`，重新顯示歡迎訊息。

## ⚠️ 使用前必要設定

ChatPanel 需要先在**設計介面 → 工具 → 設定 → Chat 設定**中完成 AI 服務設定，否則元件無法運作。

### 設定步驟

1. 開啟 **iCoder 設計介面**
2. 上方選單 → **工具** → **設定**
3. 切換到 **Chat 設定** 頁籤

### Chat 設定欄位

| 設定項 | 說明 | 必填 |
|--------|------|:----:|
| **Type** | AI 服務類型：`openai` / `openai-compatible` / `azure` / `claude` | ✅ |
| **Chat Key 設定** | 點擊編輯按鈕，設定 API Key（見下方） | ✅ |
| **Temperature** | 回應隨機度（0~2，預設 1.0，越低越精準） | 選填 |
| **Base URL** | 自訂 API 端點（僅 `openai-compatible` 需要） | 條件 |
| **Model** | 自訂模型名稱（僅 `openai-compatible` 需要，預設 gpt-4o） | 條件 |
| **Auth Header** | 自訂認證標頭名稱（僅 `openai-compatible`，預設 `Authorization`） | 條件 |
| **Auth Prefix** | 自訂認證前綴（僅 `openai-compatible`，預設 `Bearer `） | 條件 |

### Chat Key 設定

點擊 Chat Key 設定按鈕後，可設定不同 AI 服務的 API Key：

| Type | Key 類型名稱 | Key 值 |
|------|------------|--------|
| `azure` | Azure | Azure OpenAI 的 api-key |
| `claude` | Claude | Anthropic 的 x-api-key |
| `openai` | OpenAI | OpenAI 的 API Key（自動加上 `Bearer ` 前綴） |
| `openai-compatible` | OpenAICompatible | 自訂服務的 API Key |

### Prompt 設定

Chat 設定頁籤的下半部有多個 Prompt 模板，用 `{q}` 代表使用者問題、`{a}` 代表上一次回答：

| Prompt | 用途 |
|--------|------|
| **chatNewPrompt** | 新對話的提示詞（`{q}` = 使用者問題） |
| **chatRefreshPrompt** | 延續對話的提示詞（`{q}` = 問題，`{a}` = 上次回答） |
| **chatColumnPrompt** | 欄位分析用 |
| **chatTrsPrompt** | 交易回寫分析用 |
| **chatUXRequestPrompt** | ChatUX 請求分析用 |
| **chatUXTablePrompt** | ChatUX 資料表分析用 |

### 各 Type 的預設行為

| Type | API 端點 | 預設模型 | 認證方式 |
|------|---------|---------|---------|
| `azure` | config 中的 Azure 端點 | gpt-4o | Header: `api-key` |
| `claude` | `https://api.anthropic.com/v1/messages` | claude-3-5-sonnet-20241022 | Header: `x-api-key` |
| `openai` | `https://api.openai.com/v1/chat/completions` | gpt-4o | Header: `Authorization: Bearer xxx` |
| `openai-compatible` | **chatBaseUrl**（必填） | **chatModel**（或預設 gpt-4o） | **chatAuthHeader** + **chatAuthPrefix** + Key |

### 設定儲存位置

Chat 設定儲存在 `config.json`（系統設定檔），由 `ConfigHelper.GetConfig()` 讀取。所有設定為全域生效，不分方案。

## 備註

- Render 輸出完整的 Bootstrap Panel 結構：標題列（可摺疊）、聊天訊息區域、底部輸入區（含文字輸入框、檔案上傳按鈕、Send 按鈕）。
- 聊天區域高度由 `height` 屬性控制，設為 inline style `height:{Height}px;overflow-y:auto`。
- 面板支援摺疊功能，透過 Bootstrap `data-toggle="collapse"` 實現。
- `onSend` 和 `onReceive` 可攔截並修改訊息內容，回傳修改後的訊息字串。
- **未設定 Chat Key 時**，ChatPanel 送出訊息會收到空回應或錯誤。
- **openai-compatible 模式**可用於連接任何相容 OpenAI API 格式的服務（如本地 LLM、第三方代理等）。
