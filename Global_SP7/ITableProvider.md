# ITableProvider

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Provider/ITableProvider.cs` |
| 行數 | 111 行 |
| 命名空間 | `EEPGlobal.Core.Provider` |
| 繼承 | `UserDesignProvider` |

## 用途

ITable（智慧表單）Provider，載入 ITable JSON 定義並解析出 master/detail 表結構（coldefs）。

## mode 分派

| mode | 說明 |
|------|------|
| `load` | 載入指定 id 的 ITable 定義，回傳 coldefs 結構 |

## 關鍵方法

| 方法 | 說明 |
|------|------|
| `LoadITable(id)` | 讀取 `design/itable/{solution}/{id}.json`，解析 controls 取得 tableName |
| `GetITableColdefs(controls, tableNames)` | 遞迴掃描 controls，依 parentTable 關係分類 master / detail / subdetail |

## 備註

- 限制：最多 1 個 master、3 個 detail
- 分類邏輯：無 parentTable 為 master，有 parentTable 且 parent 為 master 的為 detail，否則為 subdetail
