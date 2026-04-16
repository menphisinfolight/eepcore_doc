# FLOWPHRASE

## 用途

**流程常用片語表**（Flow Phrase / Quick Text）。

FLOWPHRASE 儲存使用者在流程簽核時常用的片語/快捷文字，讓使用者可以快速選取常用意見而不需每次手動輸入。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **簽核意見快速輸入** | 在流程簽核畫面，使用者可從下拉選單快速選取常用片語 |
| **個人片語管理** | 每個使用者可維護自己的常用片語庫 |
| **共用片語** | 設定 `UserID='*'` 可建立全體共用的片語 |
| **片語排序** | 依使用者優先、時間倒序排列，最近使用的片語顯示在前面 |

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **ID** | `int identity(1,1)` PK | 自動遞增識別碼 |
| **UserID** | `nvarchar(50)` | 使用者帳號（`*` 表示全體共用） |
| **Content** | `nvarchar(max)` | 片語內容 |
| **Datetime** | `datetime` | 建立時間 |

---

## 主鍵

```
PRIMARY KEY (ID)
```

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEP.Flow/Sqls/FlowPhraseHelper.cs` | 資料庫操作類別，提供 Select/Insert/Delete 方法 |
| `EEP.Flow/API.cs` | 流程 API 層，封裝 AddPhrase/DeletePhrase/QueryPhrase 方法 |

---

## API 方法說明

| 方法 | 說明 |
|------|------|
| `QueryPhrase(pageSize, pageIndex)` | 查詢片語列表（包含個人片語 + 共用片語 `*`） |
| `AddPhrase(content)` | 新增片語（自動帶入目前使用者） |
| `DeletePhrase(id)` | 刪除指定 ID 的片語（需符合目前使用者） |

---

## 查詢邏輯

片語查詢時會同時撈取：
1. 目前使用者的個人片語（`UserID = 目前使用者`）
2. 全體共用片語（`UserID = '*'`）

排序方式：`UserID` 優先（`*` 在前），再依 `Datetime` 倒序

---

## 備註

- 片語為個人化資料，每個使用者只能看到自己的片語 + 共用片語
- 刪除片語時會驗證 `UserID`，確保只能刪除自己的片語
- 共用片語（`UserID='*'`）需由系統管理員或特定權限使用者建立
- 片語內容長度無限制（`nvarchar(max)`），可儲存長篇簽核意見
