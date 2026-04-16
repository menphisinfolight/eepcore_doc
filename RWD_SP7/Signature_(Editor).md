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

## jSignature 底層設定（隱藏屬性）

Signature 元件底層使用 **jSignature** jQuery 外掛。EEP 設計介面只暴露了 height / color / background，但 jSignature 本身有更多可設定項，可透過 JS 直接控制：

### jSignature 完整預設值

```javascript
// jSignature/src/jSignature.js line 730-744
{
    'width': 'ratio',
    'height': 'ratio',
    'sizeRatio': 4,          // height = 'ratio' 時的寬高比
    'color': '#000',          // 筆跡顏色
    'background-color': '#fff', // 背景色
    'decor-color': '#eee',    // 裝飾色（簽名線顏色）
    'lineWidth': 0,           // 畫筆粗細（0 = 自動，依壓力/速度動態調整）
    'minFatFingerCompensation': -10, // 觸控補償（負值 = 減少粗細）
    'showUndoButton': false,  // 顯示復原按鈕
    'readOnly': false,        // 唯讀
    'data': [],               // 預設資料
    'signatureLine': false    // 顯示簽名底線
}
```

### 畫筆粗細（lineWidth）

`lineWidth: 0` 表示自動模式，jSignature 會根據書寫速度動態調整筆畫粗細（快 = 細、慢 = 粗），模擬真實筆觸。設定為具體數值（如 `2`、`5`）則固定粗細。

## 前端控制範例

### 初始化時自訂 jSignature 設定

EEP 的 Signature init 時呼叫 `div.jSignature({ height, width: '100%' })`，沒有傳入 lineWidth 等參數。要自訂可以在 jSignature 初始化後修改：

```javascript
// 在 DataForm onLoad 事件中自訂畫筆粗細
function dfMaster_onLoad(row) {
    setTimeout(function() {
        var sigDiv = $('#dfMaster_簽名').next('div');
        if (sigDiv.find('.jSignature').length) {
            // 銷毀後重建，帶入自訂設定
            sigDiv.children().remove();
            sigDiv.jSignature({
                height: 115,
                width: '100%',
                lineWidth: 3,              // 固定畫筆粗細為 3
                'decor-color': '#ccc',     // 簽名底線顏色
                'signatureLine': true      // 顯示簽名底線
            });
        }
    }, 300);
}
```

### 簽名後自動填入日期時間

```javascript
function dfMaster_簽名_onChange() {
    $('#dfMaster_簽名日期').val(new Date().Format('yyyy-MM-dd hh:mm:ss'));
}
```

### 取得簽名值

```javascript
var value = $('#dfMaster_簽名').signature('getValue');
// 回傳 encodeURIComponent 後的 SVG Base64 字串
```

### 清空簽名

```javascript
$('#dfMaster_簽名').signature('reset');
```

### 重播簽名動畫

```javascript
// 需要 canReplay = true
$('#dfMaster_簽名').signature('replay');
```

### 設定唯讀

```javascript
$('#dfMaster_簽名').signature('readonly', true);  // 停用
$('#dfMaster_簽名').signature('readonly', false); // 啟用
```

### 將簽名轉為 PNG 圖片下載

```javascript
function downloadSignature() {
    var value = $('#dfMaster_簽名').signature('getValue');
    if (value) {
        var svgData = 'data:image/svg+xml;base64,' + decodeURIComponent(value);
        var img = new Image();
        img.onload = function() {
            var canvas = document.createElement('canvas');
            canvas.width = img.width;
            canvas.height = img.height;
            canvas.getContext('2d').drawImage(img, 0, 0);
            var a = document.createElement('a');
            a.href = canvas.toDataURL('image/png');
            a.download = '簽名.png';
            a.click();
        };
        img.src = svgData;
    }
}
```

## 備註

- 渲染為 `<input>` 加上 `bootstrap-signature` CSS 類別。
- 簽名值以 SVG Base64 格式儲存（`encodeURIComponent` 編碼），比 PNG 更小、可無損縮放。
- jSignature 的 `lineWidth: 0`（預設）會根據書寫速度動態調整粗細，體驗最佳。設為固定值反而會失去筆觸感。
- `minFatFingerCompensation: -10` 用於觸控螢幕，減少手指觸碰面積造成的筆畫過粗問題。
