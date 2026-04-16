# IFormCollectionExtensions

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Extension/IFormCollectionExtensions.cs` |
| 行數 | 159 行 |
| 命名空間 | `Microsoft.AspNetCore.Http` |

## 用途

`IFormCollection` 和 `IQueryCollection` 的擴充方法集，提供表單參數轉換為各種 Options 物件的便利方法。

## IFormCollectionExtensions 擴充方法

| 方法 | 說明 |
|------|------|
| `ToJObject()` | 將所有表單參數轉為 JObject |
| `GetValue<T>(key, defaultValue)` | 取得指定型別的參數值（支援 bool 特殊處理） |
| `ToQueryOptions()` | 轉為查詢選項（分頁、排序、WhereItems、TotalColumns 等） |
| `ToUpdateOptions()` | 轉為更新選項（duplicateCheck） |
| `ToExportOptions()` | 轉為匯出選項（startIndex、count、keys） |
| `ToImportOptions()` | 轉為匯入選項（startIndex、applyData、multiple、temp、tableName、beforeImport、afterImport） |

## IQueryCollectionExtensions 擴充方法

| 方法 | 說明 |
|------|------|
| `ToJObject()` | 將 QueryString 參數轉為 JObject |
| `GetValue<T>(key, defaultValue)` | 取得指定型別的 QueryString 參數值 |

## 備註

- `ToQueryOptions` 中 `page` 為 1-based 頁碼，轉換時自動減 1 計算 StartIndex
- bool 型別使用 `string.Compare(value, "True", true)` 判斷
