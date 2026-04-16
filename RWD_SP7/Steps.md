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

## 備註

- STEPS 類別與 Step 內部類別都標記了 `[DataOption]`，屬性會序列化為 `data-options` 供前端 SmartWizard 外掛初始化。
- STEPS 繼承 `RWDControl`（非 `RWDContainerControl`），子元件管理由 `Step` 集合負責。
- `GetControlTypes()` 和 `GetComponents<T>()` 會遞迴收集所有 Step 內的子元件。
- `OnNextClick` 和 `OnPrevClick` 未標記 `[DataOption]`，可能透過其他機制傳遞至前端（或需確認是否遺漏標記）。
- 渲染結構：`div.bootstrap-steps` → `ul.nav`（步驟標題列）+ `div.tab-content`（步驟內容區）。
