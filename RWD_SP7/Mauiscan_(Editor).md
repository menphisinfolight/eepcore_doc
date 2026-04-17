# Mauiscan (Editor)

> `EEPRWDTools.Core/Editors/Mauiscan.cs` — 21 行
> 繼承：`RWDEditor`
> 前端：`bootstrap.infolight.mauiapp.js` — 244 行（含註解區塊）

## 用途

**掃碼輸入編輯器**。提供 QR Code / Barcode 掃描功能，掃描結果自動填入欄位。支援兩種執行模式：

1. **Android MAUI App** — 透過原生橋接呼叫 C# 掃描功能
2. **Web / iOS** — 使用 **Html5Qrcode** 第三方套件，透過瀏覽器攝影機掃描

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 掃描實作架構

### 模式判斷

```javascript
var isAndroidApp = (window.top.invokeCSharpAction != undefined 
                    && (/Android/i.test(navigator.userAgent)));
```

- 同時具備 `invokeCSharpAction` 函式 **且** UserAgent 為 Android → Android 原生模式
- 其餘情況（iOS Safari、桌面瀏覽器）→ Web 模式（Html5Qrcode）

### Android 原生模式

按下掃描鈕時呼叫 MAUI 原生橋接：

```javascript
var paramStr = JSON.stringify({ frameId: window.location.href, resultID: id });
window.top.invokeCSharpAction("scan(" + paramStr + ")");
```

掃描完成後，MAUI App 呼叫 `window.top.SetcontrolValue()` 將結果寫回 iframe 中對應的 input。

### Web / iOS 模式（Html5Qrcode）

使用開源套件 [Html5Qrcode](https://github.com/mebjas/html5-qrcode)，流程：

1. 點擊 QR Code 按鈕 → 建立 `<div id="reader_{id}">` 區域（480×260px，黑底圓角）
2. `Html5Qrcode.getCameras()` 列出攝影機，**自動選擇後置鏡頭**（比對 label 含 `back|rear|environment|後`）
3. 啟動 `html5QrCode.start()` 以 10 fps 掃描
4. 掃描成功 → `setValue()` 填入結果 → 觸發 `change` 事件 → 自動停止
5. 超時未掃到 → 自動停止（預設 15 秒）

#### iOS 相容處理

```javascript
// 修補 video 屬性，確保 iOS Safari 能正常播放攝影機串流
$v.attr({ playsinline: 'true', webkitPlaysinline: 'true', muted: 'true', autoplay: 'true' });
```

## 隱藏設定（data 屬性）

| 設定 | 預設值 | 說明 |
|------|--------|------|
| `iosTimeoutMs` | `15000` | 掃描超時自動停止（毫秒），適用 Web/iOS 模式 |
| `qrboxSize` | `180` | 掃描框大小（px），正方形 |

HTML 用法：
```html
<input class="bootstrap-mauiscan" data-options="{iosTimeoutMs:30000, qrboxSize:250}" />
```

## 前端 JS API

```javascript
// 取值
$('#fieldId').mauiscan('getValue');

// 設值
$('#fieldId').mauiscan('setValue', 'SCAN_RESULT_123');

// 設定唯讀（同時停用掃描按鈕）
$('#fieldId').mauiscan('readonly', true);

// 取得 options
$('#fieldId').mauiscan('options');
```

## 一般瀏覽器能否使用？

**可以**，但有條件：

| 條件 | 說明 |
|------|------|
| **HTTPS** | 瀏覽器要求 HTTPS 才能存取攝影機（`getUserMedia` API 限制），localhost 除外 |
| **攝影機權限** | 使用者必須授權網頁存取攝影機 |
| **瀏覽器支援** | 現代瀏覽器皆支援（Chrome、Safari、Edge、Firefox），IE 不支援 |
| **載入 Html5Qrcode** | 需要引入 `html5-qrcode.min.js`，此檔由 `bootstrap.infolight.mauiapp.js` 的載入頁面提供 |

> 技術上，桌面瀏覽器（搭配 Webcam）和手機瀏覽器都能使用 Web 模式掃描，不限於 MAUI App。只要頁面載入了 `bootstrap.infolight.mauiapp.js` 且具備 HTTPS，非 Android App 環境會自動走 Html5Qrcode 路徑。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-mauiscan` CSS 類別
- 與 Scan 元件類似但包含完整的 Web 掃描方案（Scan 僅 Android 原生）
- 實驗性功能：`useBarCodeDetectorIfSupported: true`，在支援 BarcodeDetector API 的瀏覽器上使用原生條碼偵測，效能更好
- 掃描成功後自動觸發 `change` 事件，可搭配 DataGrid/DataForm 的 onChange 事件處理
- 停止掃描時會完整清理 video track 和 DOM 元素，避免攝影機持續佔用
