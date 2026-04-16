# Options (Editor)

> `EEPRWDTools.Core/Editors/Options.cs` — 56 行
> 繼承：`RWDEditor`

## 用途

**選項按鈕編輯器**（Radio / Checkbox）。以按鈕組方式顯示選項，支援單選和多選模式。與 Combobox 類似但以平鋪按鈕呈現。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **remoteName** | string | 命令選擇器 `[CommandEditor]` | — | 遠端資料來源名稱 |
| **fromSysParameters** | bool | 核取方塊 | false | 是否從系統參數載入選項 |
| **valueField** | string | 欄位選擇器 `[ColumnEditor]` | — | 值欄位 |
| **textField** | string | 欄位選擇器 `[ColumnEditor]` | — | 顯示文字欄位 |
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | [] | 靜態選項清單（value + text） |
| **whereItems** | List\<WhereItem\> | 集合編輯器 `[CollectionEditor]` | [] | 查詢條件 |
| **multiple** | bool | 核取方塊 | false | 是否允許多選 |
| **showTextbox** | bool | 核取方塊 `[CheckboxEditor]` | false | 是否顯示文字輸入框（「其他」選項） |
| **separator** | string | 文字 | "," | 多選時的分隔字元 |
| **mode** | OptionsMode | 列舉 | — | 選項模式 |
| **orientation** | Orientation | 列舉 | — | 排列方向 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |
| **onSelect** | string | 事件編輯器 `[ScriptEditor(value)]` | — | 選取事件 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 10750–11281 行
> jQuery 外掛名稱：`$.fn.selectoptions`

### 公開 API 方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `options()` | — | 取得元件選項物件 |
| `init(options?)` | options | 初始化元件。依 `mode` 建立不同 UI 結構（dialog: input-group + 搜尋按鈕；list: 清單區域 + 新增按鈕；其他: inline div） |
| `modal()` | — | 回傳 dialog/list 模式的 Modal jQuery 物件 |
| `div()` | — | 回傳選項容器 div |
| `textbox()` | — | 回傳 `.options-text` 文字輸入框 jQuery 物件（`showTextbox` 模式） |
| `listDiv()` | — | 回傳 list 模式的清單容器 |
| `bindEvent()` | — | 綁定各模式的互動事件（詳見下方行為說明） |
| `search()` | — | 在 Modal 中依 `.options-filter` 輸入值篩選選項（隱藏不符合的 label） |
| `changeValue(value)` | value: string | 設定值、更新文字框、觸發 `onSelect` 事件及 `$.triggerValueChanged` |
| `load()` | — | 載入選項資料。支援 `remoteName` 遠端載入、`fromSysParameters` 系統參數載入、`items` 靜態載入三種方式 |
| `loadData(data)` | data: Array | 渲染選項為 checkbox/radio 元素。依 `mode` 加入搜尋框或文字輸入框 |
| `getValues()` | — | 回傳所有已勾選項目的 value 陣列 |
| `setValues(values)` | values: Array | 依 value 陣列勾選/取消勾選 checkbox/radio |
| `addItems(values)` | values: Array | 在 list 模式中新增選項項目（含移除按鈕） |
| `getValue()` | — | 取得當前值。若有 `showTextbox` 則回傳文字框值，否則回傳隱藏 input 值 |
| `setValue(value)` | value: string | 設定值。list 模式清空後重建項目；其他模式更新勾選狀態 |
| `getWhereItems()` | — | 解析 `whereItems` 設定，透過 `$.getDefaultValue()` 動態取值後回傳查詢條件陣列 |
| `setWhere(where)` | Array 或 string | 設定查詢條件後重新呼叫 `load()` 載入選項 |
| `readonly(value)` | value: boolean | 設定唯讀狀態，停用/啟用所有互動元素 |

### 模式行為

| mode | UI 結構 | 互動方式 |
|------|---------|----------|
| `dialog` | 隱藏 input + input-group 搜尋按鈕 | 點擊/focus 按鈕後開啟 Modal，勾選後按確定將值寫回 |
| `list` | 隱藏 input + 清單區域（可新增/移除/拖曳排序） | 點擊 + 按鈕開啟 Modal 挑選；已選項目可拖曳排序或點 x 移除 |
| `button` | 隱藏 input + `btn-group` 按鈕組 | 直接點擊 toggle 按鈕選取 |
| 預設（inline） | 隱藏 input + div 內嵌 checkbox/radio | 直接點擊勾選 |

### 關鍵前端行為

- **拖曳排序（list 模式）**：支援滑鼠拖曳重新排列已選項目。mousedown 建立半透明拖曳影像，mousemove 偵測放置位置並顯示 `dropover` 樣式，mouseup 執行排序並更新值。
- **搜尋過濾（dialog/list 模式）**：Modal 頂部有搜尋框，可依選項文字即時篩選（大小寫不敏感）。
- **fromSysParameters**：啟用時自動以 `SystemTable.sysParas` 為 remoteName，以欄位名稱為 `COLUMNNAME` 查詢系統參數表。
- **separator**：多選時以指定分隔字元（預設逗號）串接/拆分值。
- **showTextbox**：啟用時在選項後方加入文字輸入框，允許使用者手動輸入自訂值。
- **值變更通知**：`changeValue()` 會自動呼叫 `$.triggerValueChanged()` 通知所屬的 DataGrid 或 Form。

## 備註

- 共用 `Combobox.Item` 類別作為靜態選項。
- 渲染為 `<input>` 加上 `bootstrap-selectoptions` CSS 類別。
