# ServiceManager

> `EEPServerTools.Core/Components/Servicemanager.cs` — 18 行
> 繼承：`Component`

## 用途

**服務管理元件**（Service Manager）。

ServiceManager 用於宣告模組中不需要登入即可呼叫的方法清單。透過 `NonlogonMethods` 屬性，設計師可以指定哪些 Server Method 允許在未登入狀態下被前端呼叫。

此元件為輕量級設定元件，僅包含一個集合屬性，無任何執行邏輯。

## JSON 設定範例

```json
{
  "type": "servicemanager",
  "id": "svcMgr",
  "nonlogonmethods": [
    { "methodname": "GetPublicData" },
    { "methodname": "HealthCheck" }
  ]
}
```

## 設計介面屬性

| 屬性 | 類型 | 設計介面 | 說明 |
|------|------|----------|------|
| **id** | string | 文字 | 元件識別碼 |
| **nonlogonmethods** | Method[] | 集合編輯器 `[CollectionEditor]` | 不需登入即可呼叫的方法清單 |

## Method — 方法定義

| 屬性 | 類型 | 說明 |
|------|------|------|
| **methodname** | string | Server Method 名稱 |

## 備註

- 此元件僅 18 行，為純設定用的空殼元件，不含任何業務邏輯或方法實作。
- 繼承自 `Component`（非 `ServerComponent`），層級較低，僅作為設定容器使用。
- `CollectionEditor` 的參數為 `("components", "servicemanager", "method", "methodName")`，表示設計介面會以方法名稱作為顯示欄位。
