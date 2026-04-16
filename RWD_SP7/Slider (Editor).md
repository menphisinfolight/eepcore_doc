# Slider (Editor)

> `EEPRWDTools.Core/Editors/Slider.cs` — 29 行
> 繼承：`RWDEditor`

## 用途

**滑桿編輯器**。提供拖拉式數值選取，適用於範圍選取場景。

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **min** | int | 數字框 `[NumberboxEditor]` | 0 | 最小值 |
| **max** | int | 數字框 `[NumberboxEditor]` | 100 | 最大值 |
| **step** | int | 數字框 `[NumberboxEditor]` | 1 | 步進值 |
| **readonly** | bool | `[Security("field")]` | false | 是否唯讀 |

## 備註

- 渲染為 `<input>` 加上 `bootstrap-slider` CSS 類別。
