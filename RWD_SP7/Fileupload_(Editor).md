# Fileupload (Editor)

> `EEPRWDTools.Core/Editors/Fileupload.cs` — 60 行
> 繼承：`RWDEditor`

## 用途

**檔案上傳編輯器**。支援檔案上傳、圖片壓縮、多檔上傳、拖放上傳、APP 拍照上傳等功能。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **folder** | string | 文字 | — | 上傳目標資料夾 |
| **showType** | FileuploadShowType | 列舉 | — | 顯示類型 |
| **dataType** | string | 下拉選項 `[ItemsEditor]` | "url" | 資料類型（url / blob） |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **editable** | bool | `[Security("field")]` | false | 是否可編輯 |
| **uploadOnSave** | bool | 核取方塊 | false | 是否在存檔時才上傳 |
| **isAutoNum** | bool | 核取方塊 | false | 是否自動編號檔名 |
| **multiple** | bool | 核取方塊 | false | 是否允許多檔上傳 |
| **dropEnable** | bool | 核取方塊 | false | 是否啟用拖放上傳 |
| **filter** | string | 多選下拉 `[ItemsEditor]` | — | 允許的副檔名（.jpg、.png、.pdf 等） |
| **compressRate** | double | 數字框 `[NumberboxEditor]` | 0 | 圖片壓縮率（0–1） |
| **sizeLimit** | double | 數字框 `[NumberboxEditor]` | 0 | 檔案大小限制（MB） |
| **appCapture** | bool | 核取方塊 | false | 是否啟用 APP 拍照 |
| **onBeforeUpload** | string | 事件編輯器 `[ScriptEditor(param, fileName)]` | — | 上傳前事件 |
| **onSuccess** | string | 事件編輯器 `[ScriptEditor(name)]` | — | 上傳成功事件 |

## 前端使用範例

```javascript
// OnBeforeUpload — 上傳前修改檔名
function dfMaster_相片_onBeforeUpload(param, fileName) {
    param.fileName = 'test.jpg'; // 修改上傳的檔名
    return true; // return false 可取消上傳
}

// OnSuccess — 上傳成功後提示
function dfMaster_相片_onSuccess(name) {
    $.alert(name + ' upload success', 'info');
}

// OnBeforeUpload — 限制檔案大小（2MB）
function dfMaster_附件_onBeforeUpload(param, fileName) {
    if (param.file && param.file.size > 2 * 1024 * 1024) {
        $.alert('檔案大小不可超過 2MB', 'error');
        return false;
    }
    return true;
}
```

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 11282–12589 行
> jQuery 插件名稱：`$.fn.fileupload`（透過 `$.createObj('fileupload', ...)` 建立）
> 輔助函式：`compressImage`（第 12549 行）、`compressImagePromise`（第 12579 行）

### HTML 結構

**多檔模式**（`multiple=true` 或 APP 拍照模式）：
```
input[type=hidden]（原始元件，hide）
input[type=file].info-file[multiple]   ← insertAfter，由 bootstrap-fileinput 接管
  .file-input                          ← fileinput 自動產生的容器
    .file-preview                      ← 檔案預覽區
    .file-caption                      ← 檔名顯示
a.file-links                          ← 多檔連結（setValue 時動態產生）
button.file-linksbutton               ← 各檔案的移除按鈕
```

**單檔模式**（`multiple=false`）：
```
input[type=hidden]（原始元件，hide）
div.input-group                        ← insertAfter
  span.form-control
    a[target=_blank]                   ← 檔案連結 / 檔名顯示
    img（showType=image 時）           ← 圖片縮圖預覽
    span.close.pull-right              ← 清除按鈕
  span.input-group-btn
    button.file-submit.glyphicon-folder-open  ← 上傳按鈕
    button.file-edit.glyphicon-pencil         ← 圖片編輯按鈕（editable=true 時）
```

### 公開 API 方法

| 方法 | 簽名 | 說明 |
|------|------|------|
| `init` | `(jq, options)` | 初始化：依 multiple 模式建立不同 UI，設定 fileinput 選項，綁定上傳事件 |
| `options` | `(jq)` → object | 取得元件設定物件 |
| `getValue` | `(jq)` → string | 取得目前值。多檔模式回傳以逗號分隔的檔名字串（含舊檔 + 新上傳檔）；單檔模式回傳 `data('value')` |
| `setValue` | `(jq, value, imageContent?)` | 設定檔案值。多檔模式清除後重設 `fileolddata`；單檔模式更新連結、縮圖。支援 blob 資料的 Base64 下載連結 |
| `getUrl` | `(jq)` → string | 取得檔案 URL。若值為完整 URL 直接回傳，否則組合 `../file?q=&f=&t=` 查詢字串 |
| `getMultiUrlLink` | `(jq, value)` | 多檔模式下產生各檔案的連結（`a.file-links`）及移除按鈕（`button.file-linksbutton`） |
| `bindEvent` | `(jq)` | 綁定上傳按鈕 click、拖放貼上（paste）事件 |
| `editImage` | `(jq)` | 開啟圖片編輯 modal（使用 jSignature），支援鉛筆/圓形/方形/文字/橡皮擦工具與多色選擇 |
| `readonly` | `(jq, value)` | 切換唯讀模式：`true` 時隱藏上傳 UI 改為純連結顯示（`showLink`）；`false` 時恢復上傳 UI |
| `showLink` | `(jq)` | 唯讀模式下渲染檔案連結或圖片預覽（blob 模式使用 Base64 data URI） |
| `uploadP` | `(jq)` → Promise | 非同步上傳（用於 `uploadOnSave` 模式）。多檔模式呼叫 `fileinput('upload')`；blob 模式以 FormData 手動 POST 至 `../file` |

### 關鍵行為

1. **上傳流程**（第 11354–11484 行）
   - `filebatchselected` 事件觸發時：
     - 若 `compressRate` 為 1.0 或 0（不壓縮）：檢查 `sizeLimit`，通過後觸發 `onBeforeUpload`，呼叫 `fileinput('upload')`。
     - 若 `compressRate` 為其他值（需壓縮）：逐一將圖片傳入 `compressImage` 壓縮後以 `$.ajax POST ../file` 上傳。使用 `$.when` 等待全部完成後 `setValue`。
   - `fileuploaded` 事件：接收伺服器回應，更新 `filedata`，觸發 `onSuccess` 回呼。

2. **圖片壓縮**（`compressImage`，第 12549–12577 行）
   - 使用 Canvas 繪製原圖，再以 `quality` 比例縮放 canvas 尺寸。
   - PNG 檔會轉為 JPEG 格式輸出。
   - 使用 `canvas.toBlob` 產生壓縮後的 Blob，品質固定為 0.9。

3. **貼上上傳**（`bindEvent` 單檔模式，第 11537–11618 行）
   - 監聽 `span.form-control` 的 `paste` 事件。
   - 若剪貼簿包含 file 類型項目，自動重命名（時間戳）後以 FormData POST 上傳。
   - 若為純文字則直接設值。

4. **圖片編輯器**（`editImage`，第 11623–12210 行）
   - 使用 `$.createModal` 建立大型 modal，內嵌 `jSignature` 手繪簽名套件。
   - 支援五種工具：鉛筆（`pencil`）、圓形（`circle`）、方形（`square`）、文字（`text`）、橡皮擦（`erase`，分大/中/小三種尺寸）。
   - 提供 8 色調色盤與 undo（復原）功能。
   - 編輯完成後將 canvas 轉為 Blob 上傳至伺服器。
   - 若無既有圖片，預設載入白色底圖（`eepcloud.infolight.com:3000/images/white.jpeg`）。

5. **Blob 模式 vs URL 模式**（`setValue` 第 11271–11310 行）
   - `dataType='blob'`：值格式為 `檔名,Base64內容`，點擊連結時呼叫 `$.base64ToUint8Array` + `$.downloadFile` 下載。
   - `dataType='url'`（預設）：值為檔案名稱，透過 `getUrl` 組合伺服器 URL。

6. **多檔移除機制**（`getMultiUrlLink`，第 12340–12392 行）
   - 每個檔案連結前方附帶移除按鈕，點擊後從 `filedata` / `fileolddata` 陣列移除對應項目，並移除 preview frame DOM。

7. **uploadOnSave 延遲上傳**（`uploadP`，第 12479–12544 行）
   - 多檔模式：監聽 `filebatchuploadsuccess` / `filebatchuploaderror` 事件，回傳 Deferred Promise。
   - Blob 模式：將 Base64 內容轉為 Blob，以 FormData 手動 POST，含 folder、compressRate、uploadKey 等參數。

### 依賴套件

- `bootstrap-fileinput`（`$.fn.fileinput`）— 多檔模式的檔案選擇與預覽 UI
- `jSignature` — 圖片編輯器的手繪功能

## 備註

- `dataType` 為 `blob` 時，檔案內容以 Base64 存入資料庫；為 `url` 時存檔案路徑。
- `compressRate` 值介於 0–1，0 表示不壓縮。
