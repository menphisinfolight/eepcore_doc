# VersionControlProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/VersionControlProvider.cs` |
| 行數 | 249 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `DesignProvider` |

## 用途

表單版本控制 Provider，提供設計資源（RWD、Server、NetFlow 等）的版本歷史查詢、版本比較、版本回滾及版本備註編輯功能。版本資料儲存在系統資料表 `menuLog`。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 依 id + menuType + solution 查詢版本清單 |
| `get` | 取得指定版本的 JSON + script 內容（version=0 為當前版本） |
| `editDesc` | 修改版本備註 |
| `remove` | 刪除指定版本 |
| `rollback` | 先備份當前版本，再還原指定版本（含 netflow XML） |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `Submit(id, type, datas)` | 提交前備份：比較當前版本與新資料，不同則寫入 menuLog |
| `Compare(data1, data2)` | 比較兩個版本的 controls + script |
| `GetVersion(id, type, version)` | 取得版本內容，版本 0 從檔案讀取，否則從 DB |
| `Rollback(id, type, version)` | 先 Submit 備份，再覆寫 .json / .js 檔案 |

## 備註

- 版本資料在 DB 中以 `to_clob` 分段儲存（每段 3900 字元），讀取時移除分段標記
- `versionControl` 設定必須為 `"true"` 才能執行版本回滾
- NetFlow 回滾時額外呼叫 `Flow.SaveToXml` 同步更新 XML 定義
