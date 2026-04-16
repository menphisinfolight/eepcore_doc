# ServerComponent

> `EEPServerTools.Core/Components/ServerComponent.cs` — 32 行
> 繼承：`Component`（抽象基底類別，位於 `EEPBase.Core`）

## 用途

**伺服器端元件基底類別**（Server Component）。

ServerComponent 是所有 EEP Server 元件的共用基底類別，繼承自 `Component`，提供對所屬 `DataModule`、`DatabaseHelper`、`ClientInfo` 的便捷存取，以及統一的事件觸發機制 `OnEvent()`。InfoCommand、UpdateComponent 等伺服器元件皆透過此類別（或其子類別）取得模組層級的資源。

## 屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Module` | DataModule | 所屬資料模組（由 `Container` 轉型而來） |
| `DbHelper` | DatabaseHelper | 資料庫操作工具（委派至 `Module.DbHelper`） |
| `ClientInfo` | ClientInfo | 當前使用者連線資訊（委派至 `Module.ClientInfo`） |

> 繼承自 `Component` 的屬性：`Id`、`CallByProc`、`Container`

## 方法

| 方法 | 簽名 | 說明 |
|------|------|------|
| `OnEvent` | `object OnEvent(string name, object[] parameters = null)` | 透過 `Module.InvokeMethod()` 觸發具名事件，自動將 `this` 作為第一個參數傳入 |

## 備註

- 這是一個薄包裝層（thin wrapper），僅 32 行，不包含業務邏輯。
- `Module` 屬性透過 `Container as DataModule` 取得，因此元件必須掛載在 DataModule 容器中才能正常運作。
- `OnEvent()` 是所有子元件觸發設計師自訂事件（如 `onBeforeInsert`、`onAfterApply`）的統一入口，實際執行由 `DataModule.InvokeMethod()` 負責反射呼叫。
