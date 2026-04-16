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

## 備註

- 每個 Item 可設定是否顯示文字輸入框（`showTextbox`）。
