# RWDPageParser (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/RWDPageParser.cs` |
| 行數 | 1103 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

將 HTML 解析為 RWD 頁面 JSON 定義。此檔案包含完整的 HTML 解析引擎（`HtmlElement`）及多種 RWD 控件解析器，用於 AI「圖片轉 RWD」功能中將 HTML 輸出轉換為 EEP RWD 頁面結構。

## 主要類別

### RWDPageParser

| 方法/屬性 | 說明 |
|-----------|------|
| `RWDPageParser(html)` | 建構時解析 HTML |
| `MasterInfo` / `DetailInfos` | 解析出的 Master/Detail 資料表結構 |
| `Controls` | 解析出的 RWD 控件列表 |
| `ToJSON()` | 輸出完整 JSON（coldefs + controls + id） |

### HtmlElement（HTML 解析引擎）

自製輕量 HTML DOM 解析器，支援：
- 標籤解析（begin/end/single/comment）
- 屬性解析（正則匹配 `attr="value"`）
- DOM 查詢：`Find(filter)`、`Children(filter)`、`Previous(filter)`、`Next(filter)`
- CSS 選擇器風格過濾：`Is("div.panel")`

### 控件解析器（RWDControlParser 子類別）

| 類別 | ElementFilter | 產出控件 |
|------|--------------|---------|
| `DataGridParser` | `table` | `DataGrid`（含 columns、toolItems） |
| `DataFormParser` | `div.panel` | `DataForm`（含 columns、editor） |
| `LayoutParser` | `div.row` | `Layout`（左 Form + 右 Grid = Query；左右 Form = Panel） |
| `LiteralParser` | `styles` | `Literal` |

### ColdefInfo / ColumnInfo

將解析出的控件轉換為 COLDEF 格式（FIELD_NAME、CHECK_NULL、NEEDBOX、FIELD_LENGTH），根據 Editor 型別對應 NEEDBOX 值（T/K/A/N/D/R/C/S 等）。

## 額外擴充

檔案底部定義了 `RWDControlsExtension`（GetParent / ToJSON）、`IToolItemExtension`（按鈕文字對應 onclick 動作）及 `ObjectExtensions`（ToJObject / ParseJObject 反射序列化）。

## 備註

- `ElementFilterAttribute` 用於標記控件解析器要匹配的 HTML 元素
- 智慧判斷 Master/Detail：第一個 DataForm 為 Master，後續 DataGrid 依是否在 DataForm 內決定為 Detail
- ID 欄位自動識別（含「编号」「編號」「ID」的欄位設為 Key）
