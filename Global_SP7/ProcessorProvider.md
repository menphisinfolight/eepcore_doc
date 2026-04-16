# ProcessorProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/ProcessorProvider.cs` |
| 行數 | 104 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

Processor（自動化處理器）執行 Provider，提供偵錯呼叫與服務預覽功能。透過組裝 `clientParam` 委託 `Adapter.Processor` 執行。

## mode 分派

| mode | 說明 |
|------|------|
| `callDebugMethod` | 偵錯執行（debugStart / debugStep / debugTo / debugStop / executeExpression） |
| `previewService` | 預覽 WebService 活動結果 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `CallProcessorMethod(id, parameters)` | 公開方法，執行指定 Processor |
| `GetClientParam()` | 組裝完整執行環境參數（含 DB 連線、匯出路徑、log 設定、chat 設定） |

## 備註

- `GetClientParam` 組裝的 `logFile` 欄位依 logTo 設定可為 `"database"`、file 路徑或空字串
- chat_setting 包含 chatType、ChatKeys、chatTemperature，供 Processor 內部使用 AI
