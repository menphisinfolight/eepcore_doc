# Layout

> `EEPRWDTools.Core/Controls/Layout.cs` — 61 行
> 繼承：`RWDControl` → `Component`

## 用途

**欄位佈局容器元件**（Layout）。

Layout 是 RWD 工具箱的佈局容器，用於將子元件以 Bootstrap Grid 的欄位（Column）方式排列。設計師可自由定義多個欄位及其寬度 class，實現響應式多欄排版。每個 Column 本身是一個容器，可放置任意子元件。

## JSON 設定範例

```json
{
  "type": "layout",
  "id": "layoutMain",
  "containerCls": "container-fluid",
  "columns": [
    {
      "columnCls": "col-xs-12 col-sm-6",
      "childrens": []
    },
    {
      "columnCls": "col-xs-12 col-sm-6",
      "childrens": []
    }
  ]
}
```

## 設計介面屬性

### Layout 主元件

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Columns** | `List<Column>` | 集合編輯器 `[CollectionEditor]` | 空集合 | 欄位集合，預設模板為兩個 `col-xs-12 col-sm-6` 欄位 |
| **ContainerCls** | string | 下拉選單 `[ItemsEditor]` | `"container-fluid"` | 外層容器 class：`container`（固定寬度）或 `container-fluid`（全寬） |

### Column 內部類別

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **ColumnCls** | string | 欄位寬度編輯器 `[ColumnClassEditor]` | `col-xs-12 col-sm-6` | Bootstrap 欄位 class，控制不同螢幕尺寸的寬度 |

## 內部類別

### Column

繼承 `RWDContainerControl`，代表一個佈局欄位。

- **Render()**：渲染 `<div>` 標籤，class 為 `ColumnCls` 的值，內部呼叫 `RenderChildren()` 渲染子元件。
- 每個 Column 是一個 `RWDContainerControl`，可透過 `childrens` 載入任意子元件。

## 備註

- Layout 類別標記了 `[DataOption]`，屬性值會序列化為 `data-options`。
- Layout 繼承 `RWDControl`（非 `RWDContainerControl`），子元件管理由 `Column` 集合負責。
- `GetControlTypes()` 和 `GetComponents<T>()` 會遞迴收集所有 Column 內的子元件。
- 渲染結構：`div.{ContainerCls}` → `div.row` → 多個 `div.{ColumnCls}`（各 Column）→ 子元件。
- `CollectionEditor` 的預設模板提供兩個等寬欄位（`col-xs-12 col-sm-6`），在手機上各占 100% 寬度，桌面上各占 50%。
