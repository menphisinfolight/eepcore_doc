# ControlProvider

| 項目 | 說明 |
|------|------|
| 路徑 | `EEPGlobal.Core/Provider/ControlProvider.cs` |
| 行數 | 277 |
| 繼承 | `DesignProvider` > `BaseProvider` |

## 用途

提供設計工具所需的控制項（Control）元資料查詢，包括屬性定義、編輯器類型、值型別、工具列項目、圖示清單、HTML 預覽、檔案管理，以及 Server 元件清單。透過反射（Reflection）從各組件的 Assembly 動態取得控制項資訊。

## ProcessRequest modes

| mode | 說明 |
|------|------|
| `getProperty` | 取得指定控制項的屬性定義 |
| `getEditor` | 取得指定分類下所有繼承自 `RWDEditor` 的編輯器類型 |
| `getValueType` | 取得指定控制項的內嵌值型別（實作 `IValueType` 的 nested type） |
| `getToolItem` | 取得指定 RWD 控制項的工具列項目 |
| `getIcon` | 取得圖示清單（Font Awesome 或 Bootstrap Glyphicon） |
| `getHtml` | 透過 `RWDPage.RenderHtml` 取得指定值的 HTML 預覽 |
| `getFile` | 瀏覽 `wwwroot` 下的檔案清單（樹狀結構） |
| `getUserFile` | 列出 `design/{fileType}/` 下的使用者檔案 |
| `removeUserFile` | 刪除 `design/{fileType}/` 下的指定檔案 |
| `getComponent` | 取得 Server 模組清單或指定模組的 InfoCommand 清單 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `GetAssembly(category)` | 依 category 取得對應 Assembly：`bootstrap` → RWDTools、`components` → ServerTools、`netflow` → FlowTools、`report` → ReportTools、`itable` → ITableTools |
| `GetProperty(category, controlType, propertyType)` | 透過 `ControlHelper` 反射取得控制項屬性陣列 |
| `GetEditor(category)` | 篩選 Assembly 中繼承自 `RWDEditor` 的型別（排除含 relation 的），回傳 camelCase 名稱 |
| `GetValueType(category, controlType)` | 取得控制項內嵌的 `IValueType` 實作型別清單 |
| `GetToolItem(category, controlType)` | 實例化 `RWDControl` 並呼叫 `GetToolItems()` |
| `GetIcons(category)` | 解析 CSS 檔案取得圖示清單：`fontawesome` 解析 `font-awesome.css`、其他解析 `bootstrap.css` |
| `GetFile(folder, fileType)` | 遍歷 `wwwroot/{folder}` 目錄，依 fileType 過濾（image/font） |
| `GetComponents(moduleName, componentType)` | 與 `SchemaProvider.GetCommands` 類似，列出 Server 模組或元件 |

## 備註

- 圖示解析使用正則表達式匹配 CSS 中的 `fa-xxx:before` 和 `glyphicon-xxx` 模式。
- `GetValueType` 對 `netflow` 分類不做 camelCase 轉換（保持原始名稱）。
- `GetFile` 支援檔案型別過濾：`image`（jpg/jpeg/bmp/png/gif/svg）、`font`（ttf/ttc）。
- `IsFileType` 使用 C# switch expression 語法。
- `getComponent` 與 `SchemaProvider.getCommand` 功能重疊，差異在於此處多了 `componentType` 參數（但實際未使用）。
