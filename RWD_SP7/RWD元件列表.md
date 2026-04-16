# EEP Core RWD 前端元件列表（SP7）

> 原始碼位置：`EEPRWDTools.Core/`
> 工具箱定義：`wwwroot/json/jquery.infolight.designer.bootstrap.toolitems.json`

## Control — 資料控制項

| 元件 | 類別 | 行數 | 說明 |
|------|------|------|------|
| [DataGrid](./DataGrid.md) | DataGrid | 663 | 資料表格（分頁、排序、編輯、匯出） |
| [TableGrid](./TableGrid.md) | TableGrid | 379 | 表格（類似 DataGrid，簡化版） |
| [DataForm](./DataForm.md) | DataForm | 564 | 資料表單（主檔編輯） |
| [DataPanel](./DataPanel.md) | DataPanel | 60 | 資料面板（唯讀顯示） |
| [DataList](./DataList.md) | DataList | 131 | 資料清單（卡片式列表） |
| [Tree](./Tree.md) | Tree | 94 | 樹狀結構 |
| [ReportViewer](./ReportViewer.md) | ReportViewer | 90 | 報表檢視器 |
| [Default](./Default.md) | Default | 64 | 預設元件 |
| [Validate](./Validate.md) | Validate | 117 | 驗證元件 |
| [FieldOnBlur](./FieldOnBlur.md) | FieldOnBlur | 44 | 欄位失焦事件 |
| [AutoSeq](./AutoSeq.md) | AutoSeq | 36 | 自動序號 |
| [Card](./Card.md) | Card | 55 | 卡片元件 |
| [Schedule](./Schedule.md) | Schedule | 96 | 行事曆/排程 |
| [ClientMove](./ClientMove.md) | ClientMove | 137 | 客戶端資料搬移 |
| [Drilldown](./Drilldown.md) | Drilldown | 80 | 鑽取/下鑽 |
| [Pivottable](./Pivottable.md) | Pivottable | 82 | 樞紐分析表 |
| [Gantt](./Gantt.md) | Gantt | 15 | 甘特圖 |
| [PromptDialog](./PromptDialog.md) | PromptDialog | 72 | 提示對話框 |
| [MenuList](./MenuList.md) | MenuList | 60 | 選單列表 |
| [Security](./Security.md) | Security | 18 | 安全元件 |
| [FLComment](./FLComment.md) | FLComment | 16 | 流程評論 |
| [ChatPanel](./ChatPanel.md) | ChatPanel | 98 | AI 聊天面板 |

## Container — 容器

| 元件 | 類別 | 行數 | 說明 |
|------|------|------|------|
| [Panel](./Panel.md) | Panel | 76 | 面板容器 |
| [Tabs](./Tabs.md) | Tabs | 175 | 分頁容器 |
| [Steps](./Steps.md) | Steps | 111 | 步驟導引容器 |
| [Layout](./Layout.md) | Layout | 61 | 版面配置容器 |

## Web — 網頁元素

| 元件 | 類別 | 行數 | 說明 |
|------|------|------|------|
| [Literal](./Literal.md) | Literal | 18 | HTML 原始碼 |
| [Label](./Label.md) | Label | 76 | 標籤文字 |
| [Image](./Image.md) | Image | 40 | 圖片 |
| [ImageList](./ImageList.md) | ImageList | 76 | 圖片列表 |
| ImageBar | ImageList | — | 圖片橫列（共用 ImageList 類別） |
| [ImageZoom](./ImageZoom.md) | ImageZoom | 60 | 圖片放大 |
| [Carousel](./Carousel.md) | Carousel | 86 | 輪播圖 |
| [Css](./Css.md) | Css | 18 | 自訂 CSS |
| [Logon](./Logon.md) | Logon | 58 | 登入元件 |
| [Menu](./Menu.md) | Menu | 114 | 選單 |
| [Breadcrumb](./Breadcrumb.md) | Breadcrumb | 52 | 麵包屑導航（SiteMap） |
| [Shoppingcart](./Shoppingcart.md) | Shoppingcart | 54 | 購物車 |
| [Counter](./Counter.md) | Counter | 28 | 計數器 |

## Chart — 圖表

| 元件 | 類別 | 行數 | 說明 |
|------|------|------|------|
| [LineChart](./LineChart.md) | LineChart | 92 | 折線圖 |
| [BarChart](./BarChart.md) | BarChart | 82 | 長條圖 |
| [PieChart](./PieChart.md) | PieChart | 66 | 圓餅圖 |
| [DonutChart](./DonutChart.md) | DonutChart | 66 | 甜甜圈圖 |
| [GeoChart](./GeoChart.md) | GeoChart | 93 | 地理圖表 |
| [SvgChart](./SvgChart.md) | SvgChart | 105 | SVG 圖表 |
| [OrgChart](./OrgChart.md) | OrgChart | 106 | 組織圖 |
| [Gauge](./Gauge.md) | Gauge | 125 | 儀表板 |

## Editor — 欄位編輯器

> 不在工具箱中，搭配 DataGrid / DataForm / TableGrid 使用。在設計介面中透過欄位屬性的「editor」設定指定。

| 元件 | 類別 | 行數 | 說明 |
|------|------|------|------|
| [Textbox](./Textbox.md) | Textbox | 74 | 文字框 |
| [Textarea](./Textarea.md) | Textarea | 36 | 多行文字框 |
| [Numberbox](./Numberbox.md) | Numberbox | 57 | 數字框 |
| [Datebox](./Datebox.md) | Datebox | 38 | 日期選擇器 |
| [Datetimebox](./Datetimebox.md) | Datetimebox | 44 | 日期時間選擇器 |
| [Dateselect](./Dateselect.md) | Dateselect | 35 | 日期下拉選擇 |
| [Timebox](./Timebox.md) | Timebox | 33 | 時間選擇器 |
| [Combobox](./Combobox.md) | Combobox | 64 | 下拉選單 |
| [Refval](./Refval.md) | Refval | 158 | 參照查詢（RefVal） |
| [Autocomplete](./Autocomplete.md) | Autocomplete | 26 | 自動完成 |
| [Submenu](./Submenu.md) | Submenu | 54 | 子選單編輯器 |
| [Options](./Options.md) | Options | 55 | 選項（Radio/Checkbox） |
| [Switch](./Switch.md) | Switch | 35 | 開關 |
| [Slider](./Slider.md) | Slider | 28 | 滑桿 |
| [Password](./Password.md) | Password | 19 | 密碼框 |
| [Fileupload](./Fileupload.md) | Fileupload | 59 | 檔案上傳 |
| [Htmleditor](./Htmleditor.md) | Htmleditor | 30 | HTML 編輯器 |
| [Multiinput](./Multiinput.md) | Multiinput | 37 | 多值輸入 |
| [Editgrid](./Editgrid.md) | Editgrid | 36 | 編輯格線 |
| [Signature](./Signature.md) | Signature | 32 | 電子簽名 |
| [Barcode](./Barcode.md) | Barcode | 21 | 條碼 |
| [Qrcode](./Qrcode.md) | Qrcode | 18 | QR Code |
| [Scan](./Scan.md) | Scan | 22 | 掃描器 |
| [Mauiscan](./Mauiscan.md) | Mauiscan | 20 | MAUI 掃描 |
| [Map](./Map.md) | Map | 24 | 地圖 |
| [Place](./Place.md) | Place | 21 | 地點選擇 |
| [Flowdesign](./Flowdesign.md) | Flowdesign | 17 | 流程設計器 |
| [Relation](./Relation.md) | Relation | 27 | 關聯設定 |
| [Creator](./Creator.md) | Creator | 21 | 建立器 |
| [Updater](./Updater.md) | Updater | 19 | 更新器 |

## 核心類別

| 檔案 | 行數 | 說明 |
|------|------|------|
| [RWDPage](./RWDPage.md) | 943 | 頁面渲染核心（產生 HTML） |
| [RWDControl](./RWDControl.md) | 138 | 控制項基底類別 |
| [RWDEditor](./RWDEditor.md) | 33 | 編輯器基底類別 |
| [RWDDef](./RWDDef.md) | 129 | 元件定義結構 |
| [HtmlString](./HtmlString.md) | 144 | HTML 字串產生工具 |
| [Enum](./Enum.md) | 124 | 列舉定義 |

---

## 元件繼承架構

```
RWDControl（控制項基底）
├── DataGrid            — 資料表格
├── TableGrid           — 表格
├── DataForm            — 資料表單
├── DataPanel           — 資料面板
├── DataList            — 資料清單
├── Tree                — 樹狀結構
├── Panel               — 面板容器
├── Tabs                — 分頁容器
├── Steps               — 步驟導引
├── Layout              — 版面配置
├── Label / Image / ... — 網頁元素
├── LineChart / ...     — 圖表
└── ...

RWDEditor（編輯器基底）
├── Textbox             — 文字框
├── Numberbox           — 數字框
├── Datebox             — 日期選擇
├── Combobox            — 下拉選單
├── Refval              — 參照查詢
├── Fileupload          — 檔案上傳
└── ...
```

## 工具箱分組對照

| 工具箱群組 | 對應分類 | 元件數 | 說明 |
|-----------|---------|--------|------|
| Control | 資料控制項 | 22 | 工具箱第一組 |
| Container | 容器 | 4 | 工具箱第二組 |
| Web | 網頁元素 | 14 | 工具箱第三組（含 ImageBar） |
| Chart | 圖表 | 8 | 工具箱第四組 |
| — | Editor（欄位編輯器） | 30 | 不在工具箱，搭配 DataGrid/DataForm 使用 |
| — | 核心類別 | 6 | 頁面渲染、基底類別、定義結構 |
| **合計** | | **84** |
