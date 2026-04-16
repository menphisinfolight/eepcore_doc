# Editgrid (Editor)

> `EEPRWDTools.Core/Editors/Editgrid.cs` — 37 行
> 繼承：`RWDEditor`

## 用途

**內嵌編輯表格編輯器**。在單一欄位中嵌入一個迷你表格，每一列可包含多個子編輯器。適用於一對多的簡易結構化資料輸入。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **columns** | List\<Column\> | 集合編輯器 `[CollectionEditor]` | [] | 欄位定義 |
| **rows** | int | 數字框 `[NumberboxEditor]` | 3 | 顯示列數 |

### Column 子類別

| 屬性 | 類型 | 說明 |
|------|------|------|
| **title** | string | 欄位標題 |
| **editor** | RWDEditor | 欄位編輯器（預設 Textbox） `[EditorOptionEditor]` |
| **values** | JArray | 預設值陣列 `[ArrayEditor]` |

## 備註

- 渲染為 `<div>` 加上 `bootstrap-editgrid` CSS 類別。
- Column 的 Editor 可以是任何 RWDEditor 子類別（Textbox、Combobox 等）。
