# NetFlowProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/NetFlowProvider.cs` |
| 行數 | 79 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

NetFlow（電子流程）設定 Provider，提供流程郵件通知範本（`flow.cfg`）的載入、儲存及重設。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 載入 `flow.cfg` 設定 |
| `save` | 儲存 `flow.cfg` 設定 |
| `reset` | 重設為預設郵件範本 |

## 備註

- 設定檔路徑：`design/config/flow.cfg`
- 預設範本包含 `emailSubject`（主旨模板）、`emailBody`（HTML 表格模板）、`emailEnable`
- 範本使用 `<%= %>` 模板語法（如 `_instance.FlowText`、`_activity.Text`）
