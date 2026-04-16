# Signature (Editor)

> `EEPRWDTools.Core/Editors/Signature.cs` — 33 行
> 繼承：`RWDEditor`

## 用途

**手寫簽名編輯器**。提供手寫簽名功能，支援自訂顏色、背景色及重播功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **height** | int | 數字框 `[NumberboxEditor]` | 120 | 簽名區高度（px） |
| **color** | string | 色彩選擇器 `[ColorEditor]` | — | 筆跡顏色 |
| **background** | string | 色彩選擇器 `[ColorEditor]` | — | 背景顏色 |
| **canReplay** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否可重播簽名過程 |
| **onChange** | string | 事件編輯器 `[ScriptEditor]` | — | 變更事件 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 12977–13216 行
> jQuery 外掛名稱：`$.fn.signature`

### 公開 API 方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `options()` | — | 取得元件選項物件 |
| `init(options?)` | options | 初始化元件：建立簽名容器 div（套用 height、background、color）、產生按鈕列（reset，可選 replay）。隱藏原始 `<input>` |
| `getValue()` | — | 取得簽名值。若存在 jSignature 實例，呼叫 `jSignature("getData", "svgbase64")` 取得 SVG Base64 編碼後以 `encodeURIComponent` 回傳；否則回傳圖片的 `data('value')` |
| `reset()` | — | 清空簽名區並重新初始化 `jSignature`。隱藏 replay 按鈕。綁定 `change` 事件，簽名修改時觸發 `onChange` |
| `replay()` | — | 重播簽名過程。解碼 SVG Base64 值，解析 SVG path 指令（M/c/l），以 Canvas 逐步繪製（30ms 間隔），動畫完成後移除暫存 Canvas |
| `setValue(value)` | value: string | 設定簽名值。有值時以 `<img>` 顯示 SVG 圖片並啟用 replay 按鈕；無值時初始化空白 jSignature 畫布 |
| `readonly(value)` | value: boolean | 唯讀時隱藏 reset 按鈕並呼叫 `jSignature('disable')`；可編輯時顯示 reset 按鈕並呼叫 `jSignature('enable')` |

### 關鍵前端行為

- **jSignature 整合**：使用 jSignature jQuery 外掛作為手寫簽名的核心繪圖引擎，輸出格式為 SVG Base64。
- **簽名儲存格式**：值以 `encodeURIComponent(svgBase64Data)` 格式儲存，還原時以 `data:image/svg+xml;base64,` 前綴顯示為 `<img>`。
- **重播動畫**：解析 SVG 的 `d` 屬性，支援 `M`（moveTo）、`c`（bezierCurveTo，相對座標）、`l`（lineTo，相對座標）三種路徑指令，以 `setInterval` 每 30ms 繪製一個路徑節點。
- **觸控支援**：簽名容器設定 `touch-action: none` 與 `-ms-touch-action: none`，防止觸控裝置上的捲動干擾手寫操作。
- **onChange 事件**：僅在 `jSignature('isModified')` 為 true 時觸發，避免初始化時誤觸。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-signature` CSS 類別。
