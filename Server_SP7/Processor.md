# Processor

> `EEPServerTools.Core/Adapter/Processor.cs` — 52 行

## 用途

**處理器介面卡**（Processor Adapter）。

Processor 是一個尚未實作的預留類別（Stub），所有方法均拋出 `NotImplementedException`。設計上作為外部處理器（Processor）的呼叫入口，由 `DataModule.CallProcessorMethod` 使用。

## 實作介面

無實作介面。

## 核心屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Context` | HttpContext | HTTP 請求上下文（唯讀） |

## 核心方法

| 方法 | 回傳型別 | 說明 |
|------|---------|------|
| `CallProcessorMethod(param)` | string | 呼叫處理器方法（未實作） |
| `CallDebugMethod(param)` | string | 呼叫除錯方法（未實作） |
| `SaveToXml(param)` | string | 儲存為 XML（未實作） |
| `PreviewService(param)` | string | 預覽服務（未實作） |

所有方法參數皆為 `dynamic` 型別。

## 備註

- 此類別目前完全未實作，僅為介面預留。實際的 Processor 呼叫邏輯可能在其他地方（如 `Flow.cs` 中的 `FlowProvider.Invoke` 透過 `DataModule.CallProcessorMethod` 實現）。
- 檔案引用了大量 using 命名空間（System.Data、System.IO、System.Net.Mail 等），但實際程式碼完全未使用，顯示為未來擴充預留。
