# Multiinput (Editor)

> `EEPRWDTools.Core/Editors/Multiinput.cs` — 38 行
> 繼承：`RWDEditor`

## 用途

**多欄輸入框編輯器**。將一個欄位拆成多個輸入區塊，值以分隔字元合併儲存。適用於電話區碼+號碼、地址分段等場景。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **items** | List\<Item\> | 集合編輯器 `[CollectionEditor]` | [] | 輸入區塊定義 |
| **count** | int | 數字框 `[NumberboxEditor]` | 2 | 區塊數量 |
| **separator** | string | 文字 | "," | 分隔字元 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

### Item 子類別

| 屬性 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| **text** | string | — | 區塊標題 |
| **showTextbox** | bool | true | 是否顯示文字輸入框 |

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 12804–12864 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 收集所有子 `<input>` 的值，以 `separator` 合併回傳 |
| `setValue(value)` | 以 `separator` 拆分值後分配至各子 `<input>`，不足的留空 |
| `readonly(bool)` | 對所有子 `<input>` 設定 `disabled` |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **兩種初始化模式**：
  - **items 模式**：依 `items` 陣列建立 `input-group-addon` 標籤和輸入框，第一個 item 的文字置於輸入框前方。
  - **count 模式**：依 `count` 數量建立等量輸入框，以 `separator` 文字作為分隔標籤。
- **包裝結構**：外層為 `input-group`，各輸入框之間以 `input-group-addon` 分隔。

## 備註

- 每個 Item 可設定是否顯示文字輸入框（`showTextbox`）。
