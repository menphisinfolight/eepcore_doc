# MSBuild

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/MSBuild.cs` |
| 行數 | 239 行 |
| 命名空間 | `EEPGlobal.Core` |

## 用途

Server 元件動態編譯引擎。將使用者撰寫的 C# 程式碼注入 `UserModule.cs`，透過 `dotnet msbuild` 編譯為 DLL。

## 主要類別

### MSBuild

| 方法 | 說明 |
|------|------|
| `Build(outputPath, codes)` | 編譯 EEPServer.Core 專案（預設） |
| `CopyAndBuild(projName, outputPath, codes)` | 複製 EEPServer.Core 專案為自訂專案後編譯 |
| `GetCode(projName)` | 讀取已有的 UserModule.cs 中使用者程式碼（#region 區塊） |

編譯流程：
1. 產生 `UserModule.cs`（CreateCodeFile），將使用者程式碼包在 `#region EEP.Core generated code` 區塊
2. 啟動 `cmd.exe` 執行 `dotnet msbuild`
3. 檢查 stdout 中的 error 行
4. 成功則將 DLL 從 `bin/Debug/{net版本}/` 搬移到 outputPath
5. 支援 net8.0 / net6.0 / net5.0

### CodeString

程式碼字串建構器，支援縮排、using、bracket 配對：

| 方法 | 說明 |
|------|------|
| `Append(value)` | 加入一行（自動依層級縮排） |
| `AppendUsing(values)` | 加入 using 宣告 |
| `AppendBeginBracket(content)` | 開始區塊 `{`（增加縮排） |
| `AppendEndBracket()` | 結束區塊 `}`（減少縮排） |

## 備註

- 編譯使用 `lock(typeof(MSBuild))` 確保同一時間只有一個編譯作業
- `CopyAndBuild` 會驗證專案名稱不等於 EEPServer，且目標目錄需有 `UserModule.cs`
- 產生的 UserModule 繼承 `DataModule`，自帶常用 using（System.Net.Http / NPOI / Newtonsoft 等）
