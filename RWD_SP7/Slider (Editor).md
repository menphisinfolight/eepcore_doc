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

## 前端行為（JavaScript）

> 原始碼：`bootstrap.infolight.js` 第 12756–12802 行

### 公開方法

| 方法 | 說明 |
|------|------|
| `getValue()` | 呼叫 `bootstrapSlider('getValue')` 取得當前值 |
| `setValue(value)` | 呼叫 `bootstrapSlider('setValue')` 設定值，同時更新旁邊的數值文字 |
| `readonly(bool)` | `true` 時呼叫 `bootstrapSlider('disable')`，`false` 時 `enable` |
| `options()` | 取得初始化選項 |

### 關鍵行為

- **bootstrapSlider 整合**：以 `min`、`max`、`step` 初始化，tooltip 設為 `hide`。
- **數值顯示**：滑桿後方插入 `<span>` 顯示當前值，`slide` 和 `slideStop` 事件同步更新。

## 備註

- 渲染為 `<input>` 加上 `bootstrap-slider` CSS 類別。
