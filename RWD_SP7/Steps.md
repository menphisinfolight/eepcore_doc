# STEPS

> `EEPRWDTools.Core/Controls/Steps.cs` — 111 行
> 繼承：`RWDControl` → `Component`

## 用途

**步驟精靈容器元件**（Steps / Wizard）。

STEPS 是 RWD 工具箱的步驟精靈容器，用於引導使用者依序完成多步驟操作。搭配 Bootstrap SmartWizard 外掛，支援多種樣式（箭頭、圓點、暗色）與切換動畫。每個步驟（Step）本身是一個容器，可放置任意子元件。

## JSON 設定範例

```json
{
  "type": "steps",
  "id": "wizardSetup",
  "style": "arrows",
  "animation": "slide-horizontal",
  "justified": true,
  "lastPageLimit": false,
  "anchorClickable": true,
  "selected": 0,
  "onNextClick": "wizardSetup_onNextClick",
  "onPrevClick": "wizardSetup_onPrevClick",
  "steps": [
    {
      "title": "基本設定",
      "childrens": []
    },
    {
      "title": "進階選項",
      "childrens": []
    }
  ]
}
```

## 設計介面屬性

### STEPS 主元件

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Steps** | `List<Step>` | 集合編輯器 `[CollectionEditor]` | 空集合 | 步驟集合，預設模板為 Step1、Step2 |
| **Style** | string | 下拉選單 `[ItemsEditor]` | `"dots"` | 步驟樣式：`default`、`arrows`（箭頭）、`dots`（圓點）、`dark`（暗色） |
| **Animation** | string | 下拉選單 `[ItemsEditor]` | `"slide-horizontal"` | 切換動畫：`none`、`fade`、`slide-horizontal`、`slide-vertical`、`slide-swing` |
| **Justified** | bool | 勾選 `[CheckboxEditor(true)]` | `true` | 是否均分步驟寬度 |
| **LastPageLimit** | bool | 勾選 `[CheckboxEditor(false)]` | `false` | 是否限制最後一頁（禁止在最後一步點擊下一步） |
| **AnchorClickable** | bool | 勾選 `[CheckboxEditor(true)]` | `true` | 是否允許點擊步驟標題直接跳轉 |
| **Selected** | int | 數字框 `[NumberboxEditor(0)]` | `0` | 預設選中的步驟索引（從 0 開始） |
| **OnNextClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊下一步事件，參數：`index` |
| **OnPrevClick** | string | 腳本編輯器 `[ScriptEditor]` | — | 點擊上一步事件，參數：`index` |

### Step 內部類別

| 屬性 | 類型 | 設計介面 | 預設值 | 說明 |
|------|------|----------|--------|------|
| **Title** | string | 文字 `[Localization]` | — | 步驟標題（支援多語系） |

## 內部類別

### Step

繼承 `RWDContainerControl`，標記 `[DataOption]`，代表單一步驟。

- **RenderHeader()**：渲染步驟標題導覽（`<li>` → `<a class="nav-link">`）。連結指向 `#{Id}`。
- **Render()**：渲染步驟內容（`div.tab-pane`，帶 `role="tabpanel"`），內部呼叫 `RenderChildren()` 渲染子元件。
- 每個 Step 的 `Id` 由 STEPS 渲染時自動以 `{STEPS.Id}_{index}` 格式指派。

## 前端行為（JavaScript）

> 原始碼位置：`bootstrap.infolight.js` 第 10622–10748 行
> jQuery 外掛名稱：`$.fn.steps`

### 初始化流程

1. 解析 `data-options`，呼叫 jQuery SmartWizard 外掛 (`smartWizard()`) 進行初始化。
2. 傳入設定包含：`keyNavigation: false`（關閉鍵盤導覽）、`anchorClickable`、`theme`（對應 `style`）、`transition`（對應 `animation`，速度固定 400ms）、`justified`、`selected`。
3. 語系文字（下一步 / 上一步按鈕）取自 `$.fn.locale.next` / `$.fn.locale.previous`。

### 事件綁定（bindEvent）

- **下一步按鈕（`.sw-btn-next`）**：點擊時觸發 `onNextClick` 回呼，傳入「前一步索引」（當前索引 - 1）。若到達最後一步且 `lastPageLimit` 為 true，隱藏上一步按鈕並移除所有步驟的 `done` 狀態。
- **上一步按鈕（`.sw-btn-prev`）**：點擊時觸發 `onPrevClick` 回呼，傳入「後一步索引」（當前索引 + 1）。

### 與表單（Form）的整合

當 Steps 位於 `.bootstrap-form` 內時，自動啟用表單精靈模式：

1. 隱藏表單工具列（`.dataform-toolitem`）。
2. 在工具列新增一個「確定」按鈕（`.sw-btn-ok`），點擊後呼叫 `bindingForm.form('submit')` 提交表單。
3. 監聽 `stepContent` 事件：
   - 最後一步時，隱藏「下一步」、顯示「確定」；若 `lastPageLimit` 為 true，同時隱藏「上一步」並禁用錨點點擊。
   - 非最後一步時，顯示「下一步」、隱藏「確定」。
4. 表單送出成功後（`onApplied`）：自動跳轉到下一步（`smartWizard('next')`）並新增一列（`form('insert_row')`）。
5. 若未自訂 `onApplied`，預設顯示「儲存成功」訊息，並關閉目前頁籤。

### 方法

| 方法 | 說明 |
|------|------|
| `options` | 取得元件設定 |
| `init` | 初始化 SmartWizard 並綁定事件 |
| `bindEvent` | 綁定上一步/下一步/確定按鈕事件 |
| `resize` | 重新調整步驟大小（透過重設 `current_index` 強制重新渲染當前步驟） |

## 備註

- STEPS 類別與 Step 內部類別都標記了 `[DataOption]`，屬性會序列化為 `data-options` 供前端 SmartWizard 外掛初始化。
- STEPS 繼承 `RWDControl`（非 `RWDContainerControl`），子元件管理由 `Step` 集合負責。
- `GetControlTypes()` 和 `GetComponents<T>()` 會遞迴收集所有 Step 內的子元件。
- `OnNextClick` 和 `OnPrevClick` 未標記 `[DataOption]`，可能透過其他機制傳遞至前端（或需確認是否遺漏標記）。
- 渲染結構：`div.bootstrap-steps` → `ul.nav`（步驟標題列）+ `div.tab-content`（步驟內容區）。
