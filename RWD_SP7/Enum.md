# Enum

> `EEPRWDTools.Core/Enum.cs` — 124 行

## 用途

**RWD 元件列舉定義**（Enumerations）。

集中定義 EEPRWDTools.Core 模組中所有控制項使用的列舉型別，涵蓋表單模式、查詢模式、對齊方式、統計方式、驗證樣式及各種控制項專屬選項。

## 列舉一覽

| 列舉 | 值 | 說明 |
|------|-----|------|
| `FormMode` | Dialog, Panel, Tab | 表單顯示模式（對話框 / 面板 / 頁籤） |
| `QueryMode` | Dialog, Panel, Fuzzy | 查詢面板模式（對話框 / 面板 / 模糊查詢） |
| `DrilldownMode` | Tab, Dialog, Window, Table | 明細展開模式（頁籤 / 對話框 / 新視窗 / 表格內） |
| `ToolItemPosition` | Top, Bottom | 工具列按鈕位置 |
| `IconAlign` | Left, Right | 圖示對齊方向 |
| `TextAlign` | Left, Right | 文字對齊方向 |
| `Total` | None, Sum, Max, Avg, Count | 欄位合計方式 |
| `TotalMode` | Page, All | 合計範圍（當頁 / 全部） |
| `ValidateStyle` | Hint, Dialog, Timely | 驗證提示方式（提示文字 / 對話框 / 即時驗證） |
| `BarcodeFormat` | Code39, Code128, Ean13, Ean128 | 條碼格式 |
| `DateboxDataType` | Datetime, Varchar8 | 日期欄位資料型態（DateTime / 8 位字串 yyyyMMdd） |
| `DatetimeboxView` | Hour, Day, Month | 日期時間選擇器檢視層級 |
| `FileuploadShowType` | Link, Image | 檔案上傳顯示方式（連結 / 圖片預覽） |
| `OptionsMode` | Checkradio, Button, Dialog, List | 選項呈現方式（勾選 / 按鈕 / 對話框 / 清單） |
| `Orientation` | Horizontal, Vertical | 排列方向（水平 / 垂直） |
| `MapValueType` | Latlng, Address, Current, CurrentAddress | 地圖值類型（經緯度 / 地址 / 目前位置 / 目前地址） |
| `PlaceValueType` | Address, Latlng | 地點值類型（地址 / 經緯度） |
| `RefvalQueryMode` | None, Fuzzy | 參照值查詢模式（無 / 模糊） |

## 備註

- 所有列舉皆位於 `EEPRWDTools.Core` 命名空間，供各控制項屬性直接引用。
- `FormMode`、`QueryMode`、`DrilldownMode` 是影響頁面佈局行為的核心列舉。
- `Total` 和 `TotalMode` 搭配使用，控制 DataGrid 等表格控制項的合計功能。
