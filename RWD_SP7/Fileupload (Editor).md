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

## 備註

- `dataType` 為 `blob` 時，檔案內容以 Base64 存入資料庫；為 `url` 時存檔案路徑。
- `compressRate` 值介於 0–1，0 表示不壓縮。
