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

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 15761–15884 行
> jQuery 外掛名稱：`$.fn.editgrid`

### 初始化流程

1. 解析 `data-options`，設定容器 CSS（`height: auto`、`padding: 0`、`border: none`）。
2. 根據 `columns` 建立 `<table class="bootstrap-datagrid table editgrid-table table-hover table-striped table-bordered table-condensed">`。
3. 表頭（`<thead>`）依欄位標題產生，每欄寬度均分（`100% / 欄數`）。
4. 表身（`<tbody>`）依 `rows` 數（預設 3）產生列數，每格以 `$.fn.datagrid.defaults.editors` 建立對應的編輯器。
5. 若 column 有 `values` 且該列索引在範圍內，改用 `label` 類型顯示預設值。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化表格與各格編輯器 |
| `getValue` | 遍歷所有列與格，以各 editor 的 `getValue` 取值，回傳二維陣列的 JSON 字串 |
| `setValue(value)` | 接收 JSON 字串（二維陣列），解析後以各 editor 的 `setValue` 填值。若 `value` 為空，清空所有格的值 |
| `readonly(value)` | 設定所有格的唯讀狀態。會優先保留各格自身 `options.readonly` 設定，最終以 `setReadonly(readonly \|\| value)` 套用 |

### 資料格式

`getValue` / `setValue` 使用 JSON 二維陣列，例如：
```json
[["A1","B1","C1"],["A2","B2","C2"],["A3","B3","C3"]]
```

## 備註

- 渲染為 `<div>` 加上 `bootstrap-editgrid` CSS 類別。
- Column 的 Editor 可以是任何 RWDEditor 子類別（Textbox、Combobox 等）。
