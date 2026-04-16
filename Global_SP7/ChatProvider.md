# ChatProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/ChatProvider.cs` |
| 行數 | 1814 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

整合 AI 聊天功能的 Provider，支援多種 AI 服務（Azure OpenAI、Claude、OpenAI、openai-compatible），提供設計端的 AI 輔助功能，包含文字問答、圖片轉程式碼、流程圖生成、欄位比對、測試資料產生、SQL 輔助等。

## mode 分派

| mode | 說明 |
|------|------|
| `send` | 基本問答，使用 `chatNewPrompt` / `chatRefreshPrompt` |
| `columns` | 欄位問答（chatColumnPrompt） |
| `trs` | 轉換問答（chatTrsPrompt） |
| `chatUXRequest` | UX 需求問答 |
| `chatUXTable` | UX 表格問答 |
| `chatUXColumn` | UX 欄位問答 |
| `chatTest` | 直接發送 prompt 測試 |
| `getChatConfig` | 取得所有 chat prompt 設定 |
| `saveChatUXAccess` | 儲存 chatUXAccess 至 `global.cfg` |
| `imageToHtml` | 圖片轉 HTML（ITable 頁面） |
| `imageToReport` | 圖片轉報表 JSON |
| `jsonToFlow` | JSON 轉流程定義 |
| `tableToFormula` | 表格轉公式 |
| `convertHtml` | HTML 轉 JSON（支援 Rwd / ITable / RTable） |
| `columnMatch` | 欄位名稱與資料庫欄位比對（chat 或 coldef） |
| `reportColumnMatch` | 報表欄位比對 |
| `createTestData` | AI 產生測試資料 |
| `assistantChat` | OpenAI Assistant 或 compatible 模式對話 |
| `uploadAssistant` | 上傳 prompt 建立 Assistant |
| `exportExcel` | 匯出 RTable Excel |
| `sqlHelper` | SQL 輔助，依資料庫類型產生 SQL |

## AI 服務支援

透過 `GetConfigValues` 根據 `chatType` 組裝 API 參數：

| chatType | 說明 |
|----------|------|
| `azure` | Azure OpenAI，URL 從 `chatUrl` 設定讀取 |
| `claude` | Anthropic Claude API |
| `openai` | OpenAI 官方 API（含 Assistant 模式） |
| `openai-compatible` | 自訂 OpenAI 相容端點，需設 `chatBaseUrl` |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `sendChat(prompt, showTimeInfo)` | 核心發送方法，組裝 messages、呼叫 API、記錄 log |
| `GetResponse(chatType, ...)` | HTTP 呼叫 AI，解析 claude / openai 回應格式，抽取 JSON |
| `GetConfigValues(config, assistantID)` | 根據 chatType 組裝 URL / model / key / header |
| `ScaleImage(bytes, imageType)` | 圖片縮放至 1400px 以內、壓縮至 5MB 以下 |
| `imageToHtml(...)` | 將圖片送 AI 辨識，回傳 JSON 並透過 `HtmlParser` 轉 HTML |
| `ImageToFlow(file, promptText)` | 上傳檔案（圖片或文字）送 AI 辨識為流程 JSON，再以 `FlowParser` 轉換 |
| `assistantChat(...)` | OpenAI Assistant Thread/Run 模式或 openai-compatible 直接 chat |
| `columnMatch(controls, matchType)` | 比對控件欄位名與資料庫欄位（coldef 或 AI chat） |
| `convertHtml(html, json, codeType, setting)` | 委託 `RWDPageParser` / `ITableParser` / `RTableParser` 轉換 |

## 備註

- 使用 `SkiaSharp` 處理圖片縮放，非 Windows GDI+
- `_openaiCompatiblePrompts` 為靜態字典，在 openai-compatible 模式下暫存各類 prompt
- Assistant 模式使用 OpenAI Assistants v2 API（Thread → Message → Run → 輪詢結果）
- Log 同時寫入 `LogHelper.DebuggerLog` 和 `design/config/logs/` 文字檔
