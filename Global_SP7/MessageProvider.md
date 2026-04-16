# MessageProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/MessageProvider.cs` |
| 行數 | 151 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

多國語系訊息管理 Provider，提供系統訊息（message.cfg）的載入、搜尋與儲存功能。支援 default、zh-cn、zh-tw、ja-jp、ko-kr、it1、it2 等語系。

## mode 分派

| mode | 說明 |
|------|------|
| `get` | 取得訊息列表（支援關鍵字篩選與分頁） |
| `save` | 儲存訊息（覆寫或合併） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetMessage(isUserDefined, filter, options)` | 讀取訊息檔，支援跨語系搜尋與分頁 |
| `SaveMessage(isUserDefined, datas)` | 儲存訊息，按 key 排序後寫入檔案 |

## 備註

- `isUserDefined = true` 時讀取方案級 `message.cfg`，否則讀取系統級 `MessageHelper.FILE_PATH`
- 儲存時系統級訊息會先讀取既有內容再合併，方案級則直接覆寫
- 訊息 key 儲存前會以 `StringComparer.Ordinal` 排序
