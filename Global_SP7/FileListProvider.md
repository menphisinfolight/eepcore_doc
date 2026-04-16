# FileListProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/FileListProvider.cs` |
| 行數 | 205 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

提供設計工具的檔案清單樹狀結構，用於前端檔案瀏覽器顯示 Word、Excel 及其他選單類型的檔案列表。支援依群組（group）分層顯示。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `get` | 取得指定資料夾的檔案清單（樹狀結構 JArray） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetFileList(folder)` | 主要路由方法：`folder` 為空時回傳根目錄（Word、Excel 資料夾）；非空時依類型取得檔案節點 |
| `FolderItem(id, text)` | 建立資料夾節點 JObject（含 iconCls、state=closed） |
| `GetMenuNodes(type, group, commandName, nameField)` | 從系統資料表（`SYS_DOCFILES` / `SYS_XLSFILES`）查詢檔案列表，回傳樹狀節點 |
| `GetFileList(folder, group)` | 從 `MenuProvider.GetMenuNodes` 取得指定目錄下的 `.json` 檔案列表 |

## 備註

- 根目錄固定包含 `word` 和 `excel` 兩個資料夾節點。
- 支援的檔案類型路由：`word`、`wordPdf` 查 `SYS_DOCFILES`；`excel` 查 `SYS_XLSFILES`；`wordExcel` 查 `SYS_FILES`；其他類型走 `GetFileList` 讀取實體檔案。
- 群組機制：透過 `MenuProvider.GetMenuGroup()` 取得群組定義，將檔案歸類到對應群組。`group` 為空時回傳群組資料夾節點，非空時回傳該群組下的檔案。
- `GetConfig()` 方法取得設定檔並附加公鑰（`EncryptHelper.PublicKey`），但目前未在 `ProcessRequest` 中使用。
- 節點的 `editable` 屬性由 `ClientInfo.Dev` 控制（開發模式才可編輯）。
