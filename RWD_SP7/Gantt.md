# Gantt

> `EEPRWDTools.Core/Controls/Gantt.cs` — 15 行
> 繼承：`RWDControl` → `Component`

## 用途

**甘特圖元件**（Gantt）。

Gantt 為需要授權的功能元件，目前僅為佔位實作。Render 時輸出「請先獲取授權再使用Gantt功能」提示文字。

## 前端行為（JavaScript）

> 來源：`bootstrap.infolight.js` 第 16975–17817 行（約 843 行）
> C# 僅 15 行佔位（未授權提示），**所有甘特圖功能皆由前端 JS 實現**。

### 元件註冊

以 `$.fn.gantt = $.createObj('gantt', {...})` 註冊為 jQuery 插件，底層使用 **dhtmlxGantt** 函式庫（`gantt.*` API）。

### 可設定屬性（options）

以下屬性從 HTML `data-options` 或程式傳入：

| 屬性 | 類型 | 說明 |
|------|------|------|
| **remoteName** | string | 遠端資料命令名稱（透過 `$.loadData` 載入） |
| **whereStr** | string | WHERE 條件字串 |
| **whereItems** | Array | WHERE 條件陣列 |
| **rowId** | string | 資料列主鍵欄位名 |
| **textColumn** | string | 顯示文字欄位名 |
| **startDate** | string | 開始日期欄位名 |
| **endDate** | string | 結束日期欄位名 |
| **duration** | string | 工期欄位名（與 endDate 二擇一） |
| **parentID** | string | 父節點 ID 欄位名（樹狀結構） |
| **progress** | string | 進度欄位名（0–1） |
| **actuallyStart** | string | 實際開始日期欄位名（基線比較用） |
| **actuallyEnd** | string | 實際結束日期欄位名（基線比較用） |
| **panelHeight** | number | 甘特圖面板高度（px） |
| **scale** | string | 時間刻度：`"day"`（預設）、`"week"`、`"month"` |
| **readonly** | bool | 是否唯讀模式 |
| **autoSave** | bool | 是否在每次新增/修改/刪除後自動儲存 |
| **autoScheduling** | bool | 是否啟用自動排程（嚴格模式） |
| **showGrid** | bool | 是否顯示左側列表面板（`false` 時 `grid_width=0`） |
| **showStartDateColumn** | bool | 列表面板是否顯示開始日期欄 |
| **showDurationColumn** | bool | 列表面板是否顯示工期欄 |
| **showToday** | bool | 是否顯示今日標記線 |
| **textCaption** | string | 列表面板文字欄標題（覆蓋預設） |
| **startDateCaption** | string | 列表面板開始日期欄標題 |
| **durationCaption** | string | 列表面板工期欄標題 |
| **titleFormatter** | function | 自訂任務 tooltip 格式化函數 `function(row)` |

### 回呼事件

| 事件 | 時機 | 說明 |
|------|------|------|
| **onBeforeLoad** | `load()` 發 AJAX 前 | `onBeforeLoad.call(this, opts)`，可修改 options |
| **onLoad** | `load()` 取回資料後 | `onLoad.call(this, data.rows)`，取得原始資料列 |

### 公開方法

| 方法 | 參數 | 說明 |
|------|------|------|
| `init` | options? | 初始化：設定高度、綁定工具列按鈕、呼叫 `renderGantt` 及 `load` |
| `load` | — | 從 `remoteName` 載入資料，自動欄位映射後呼叫 `loadData` |
| `renderGantt` | — | 配置 dhtmlxGantt 並呼叫 `gantt.init()`（詳見下方） |
| `loadData` | datajson | 清除既有資料後 `gantt.parse(datajson)`，初始化 rowData 變更追蹤 |
| `save` | — | 將 inserted/updated/deleted 資料透過 `$.updateData` 寫回伺服器 |
| `setWhere` | where | 設定 whereStr 或 whereItems，然後 `refresh` |
| `refresh` | — | 重新 `load` 資料 |
| `clear` | — | `gantt.clearAll()` |
| `refreshData` | data | 清除後重新 `gantt.parse(data)` |
| `undo` | — | `gantt.undo()` |
| `redo` | — | `gantt.redo()` |
| `exportToMSProject` | — | 匯出為 MS Project XML（`.xml`） |
| `exportToExcel` | — | 匯出為 Excel（`.xlsx`） |
| `exportToPDF` | — | 匯出為 PDF（`.pdf`） |
| `importFromExcel` | — | 從 Excel 匯入（開啟檔案選擇對話框） |
| `importFromMsProject` | — | 從 MS Project 匯入（尚未實作，空函數） |

### 資料載入與欄位映射（load 方法）

`load()` 透過 `$.loadData(remoteName, ...)` 取得資料後，將伺服器欄位映射為 dhtmlxGantt 所需格式：

- `id` ← `row[opts.rowId]`
- `start_date` ← `row[opts.startDate]`（轉為 `yyyy-mm-dd` 格式）
- `text` ← `row[opts.textColumn]`
- `parent` ← `row[opts.parentID]`
- `progress` ← `row[opts.progress]`（如有設定）
- `duration` ← `row[opts.duration]`（如有設定）
- `end_date` ← `row[opts.endDate]`（當 duration 未設定時使用）
- `actuallyStart` / `actuallyEnd` ← 實際日期（基線）

載入後建立 `gantt.createDataProcessor`，追蹤 task 的 create / update / delete 操作，並在 `autoSave` 為 true 時自動呼叫 `save()`。

### 甘特圖渲染設定（renderGantt 方法）

#### 時間刻度

內建 8 種刻度配置（`scaleConfigs[0..7]`）：

| 索引 | 單位 | 主刻度 | 副刻度 |
|------|------|--------|--------|
| 0 | minute | 時（%H） | 分（%H:%i） |
| 1 | hour | 日（%M%j） | 時（%H:%i） |
| 2 | **day**（預設） | 月（%F） | 日（%j） |
| 3 | week | 月（%F） | 週範圍（M/d-M/d） |
| 4 | month | 年（%Y） | 月（%M） |
| 5 | quarter | 年（%Y） | 季範圍（M-M） |
| 6 | year | 年（%Y） | 5 年區間 |
| 7 | decade | 10 年區間 | 100 年區間 |

根據 `opts.scale` 值選用 day / week / month 其中之一（索引 2、3、4）。

#### 左側列表欄位

當 `showGrid` 不為 `false` 時，左側面板包含：

| 欄位 | 寬度 | 備註 |
|------|------|------|
| text（樹狀展開） | 150px | 可 resize |
| start_date | 150px | 由 `showStartDateColumn` 控制顯示 |
| duration | 70px | 由 `showDurationColumn` 控制顯示 |
| add（新增按鈕） | 44px | 目前固定隱藏 |

#### 基線（Baseline）功能

- 透過 `gantt.addTaskLayer` 繪製基線條（class `baseline`），顯示在任務條下方。
- Lightbox 編輯器包含基線日期區段（`duration_optional` 類型），可選填 `actuallyStart` / `actuallyEnd`。
- 若計畫結束日 < 實際結束日，任務標記 `overdue` 樣式，右側顯示「超時：N 天」文字。
- `onTaskLoading` 事件中將基線日期字串解析為 Date 物件。

#### 其他渲染設定

- `xml_date` 格式：`%Y-%m-%d %H:%i:%s`
- `task_height`：16px、`row_height`：40px
- Lightbox 區段：description（textarea）、time（duration）、baseline（duration_optional）
- `onDataRender` 事件中套用 `titleFormatter`（如有設定）至每個任務列的 tooltip。
- `showToday` 為 true 時，以 `gantt.addMarker` 加入今日標記線（css class `today`）。

### 儲存機制（save 方法）

- 維護 `rowData = { inserted: [], updated: [], deleted: [] }` 追蹤變更。
- 儲存時透過 `$.updateData(remoteName, datas, ...)` 批次寫回。
- 新增資料回傳後，記錄 `insertedID`（oldid → newid 映射），後續 update/delete 會自動替換暫時 ID。

### 匯入 Excel 流程（importFromExcel 方法）

1. 開啟 modal 對話框（`#ganttImportFile`），提供檔案上傳表單（`.xlsx` / `.xls`）。
2. 使用 `gantt.importFromExcel` 上傳至 `../excelfile` 端點。
3. 匯入完成後彈出欄位對應對話框（modalbox），使用 `<select>` 讓使用者將 Excel 欄位映射至甘特圖欄位。
4. 確認後根據映射轉換資料，以 WBS 編號推導層級關係（`id` = WBS、`parent` = 去除最後一段的 WBS）。
5. 清除現有資料並載入匯入結果。

### 工具列按鈕

`init` 時綁定 `.gantt-toolitem .btn` 的 click 事件：
- 若 `onclick` 名稱對應 `$.fn.gantt.methods` 中的方法，則呼叫該方法。
- 否則透過 `$.callFunction` 呼叫自訂函數。

## 備註

- 無任何可設定屬性，無 `[DataOption]` 標記。
- 類別宣告為 `class`（非 `public class`），存取範圍為 internal。
- Render 輸出 `<span>請先獲取授權再使用Gantt功能</span>`。
