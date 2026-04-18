# EEP Server 模組的建置與偵錯機制

> EEP 的 Server 端設計有獨特的**自動建置機制**。理解它能避免幾個常見誤區：為什麼寫程式碼不用加 `using`、為什麼只能 debug 最後存檔的那個模組、要怎麼在 VS Code 開發、怎麼加入新的 namespace 引用。

## TL;DR（快速結論）

1. **存檔 = 自動建置 DLL** — EEP 設計介面按「存檔」觸發 `MSBuild.Build`，把程式碼包進 `UserModule : DataModule` → 用 `dotnet msbuild` 編譯 → 把 DLL 搬到對應資料夾
2. **不可寫 `using` / `namespace` / `class`** — 框架已自動包好，寫了會語法錯誤
3. **只能 Debug 最後存檔的模組** — 所有 Server 模組**共用同一份 `EEPServer.Core/UserModule.cs`**，每次存檔被覆寫
4. **VS Code 開發要手動貼回 EEP** — 沒有「在 VS Code 直接編輯就存進 EEP」的整合
5. **加新引用要改 `MSBuild.cs`** — `code.AppendUsing(...)` 裡面加新 namespace 並重 build EEPGlobal.Core

## 建置流程（存檔觸發）

### 1. 使用者按下存檔

在 EEP 設計介面的 Server 模組 / Report / Flow 等編輯頁，按「存檔」送出：
```
POST /design/ + { type: "menuProvider", mode: "saveMenu", id: "訂單模組", script: "public object foo() {...}" }
```

### 2. `MenuProvider.SaveMenu()` 流程
> 檔案：`EEPGlobal.Core/Provider/MenuProvider.cs` L1006-1037

```csharp
if (datas.script != null) {
    if (!string.IsNullOrEmpty(datas.script.ToString())) {
        if (menuType == "server") {
            // 2a. 把使用者寫的 C# 程式碼存成 .cs 檔
            using (StreamWriter writer = new StreamWriter(
                Path.Combine(menuDir, $"{id}.cs"), false, new UTF8Encoding()))
            {
                writer.Write(datas.script.ToString());
            }

            // 2b. 呼叫 MSBuild.Build 編譯並產出 DLL
            MSBuild.Build(
                Path.Combine(menuDir, $"{id}.Core.dll"),   // 目標 DLL 路徑
                datas.script.ToString()                     // 使用者程式碼
            );
        }
    }
}
```

**`menuDir` 位置**：`design/server/{ClientInfo.Solution}/`（由 `GetMenuDir("server")` 決定，最終路徑在 `BaseProvider.cs:69`）

### 3. `MSBuild.Build()` 內部流程
> 檔案：`EEPGlobal.Core/MSBuild.cs` L21-126

```csharp
public static void Build(string outputPath, string codes)
{
    var projName = "EEPServer";
    var projDir = $"{ Path.GetDirectoryName(CurrentDirectory)}/{projName}.Core";
    //         → 例如：{EEPWebClient.Core 的父目錄}/EEPServer.Core

    Build(projName, projDir, outputPath, codes);
}

private static void Build(string projName, string projDir, string outputPath, string codes)
{
    lock (typeof(MSBuild))   // 多使用者同時存檔會排隊
    {
        // (A) 把使用者程式碼包成完整 .cs
        var code = CreateCodeFile(codes);

        // (B) 寫入 UserModule.cs（★ 共用，會被覆寫 ★）
        using (var writer = new StreamWriter(
            Path.Combine(projDir, "UserModule.cs"), false, new UTF8Encoding()))
        {
            writer.Write(code.ToString());
        }

        // (C) 執行 dotnet msbuild
        var p = new Process { ... WorkingDirectory = projDir, FileName = "cmd.exe", ... };
        p.Start();
        p.StandardInput.WriteLine("chcp 65001");
        p.StandardInput.WriteLine("dotnet msbuild");
        p.StandardInput.WriteLine("exit");
        var result = p.StandardOutput.ReadToEnd();

        // (D) 檢查編譯錯誤
        var errorLines = result.Split('\n').Where(line => line.Contains(" error ")).ToList();
        if (errorLines.Count > 0) throw new EEPException(...);

        // (E) 搬 DLL：EEPServer.Core/bin/Debug/net8.0/EEPServer.Core.dll → {id}.Core.dll
        File.Move(
            Path.Combine(projDir, $"bin/Debug/net8.0/{projName}.Core.dll"),
            outputPath,      // design/server/{Solution}/{id}.Core.dll
            true);
    }
}
```

### 4. `CreateCodeFile()` — 包裝使用者程式碼
> 檔案：`EEPGlobal.Core/MSBuild.cs` L128-162

這是**為什麼不能寫 `using` / `namespace` / `class`** 的真正原因：

```csharp
private static CodeString CreateCodeFile(string codes)
{
    var code = new CodeString();

    // 自動注入 18 個 namespace（使用者看不到、不需寫）
    code.AppendUsing(new string[] {
        "EEPBase.Core",
        "EEPBase.Core.Utility",
        "EEPServerTools.Core",
        "EEPServerTools.Core.Components",
        "Newtonsoft.Json.Linq",
        "System",
        "System.Collections.Generic",
        "System.Data",
        "System.Linq",
        "System.Net.Http",
        "System.Threading.Tasks",
        "System.IO",
        "System.Net",
        "System.Text",
        "System.Text.Json",
        "System.Web",
        "NPOI.SS.UserModel",
        "NPOI.XSSF.UserModel"
    });

    // 自動包 namespace + class
    code.AppendBeginBracket("namespace EEPServer.Core");
    code.AppendBeginBracket("public class UserModule: DataModule");
    code.Append("public UserModule(ClientInfo clientInfo, string name, IEnumerable<Component> components) : base(clientInfo, name, components) { }");

    // 使用者程式碼插在 #region 內（在 class 裡面）
    code.Append($"#region {REGION_STR}\n");
    code.Append(codes.Split('\n'));
    code.Append($"#endregion\n");

    code.AppendEndBracket();   // class
    code.AppendEndBracket();   // namespace
    return code;
}
```

**產出檔案 `EEPServer.Core/UserModule.cs` 的結構**：

```csharp
using EEPBase.Core;
using EEPBase.Core.Utility;
using EEPServerTools.Core;
using EEPServerTools.Core.Components;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
using System.IO;
using System.Net;
using System.Text;
using System.Text.Json;
using System.Web;
using NPOI.SS.UserModel;
using NPOI.XSSF.UserModel;

namespace EEPServer.Core {
    public class UserModule : DataModule {
        public UserModule(ClientInfo clientInfo, string name, IEnumerable<Component> components)
            : base(clientInfo, name, components) { }

        #region EEP.Core generated code

        // ← 使用者在設計介面寫的程式碼插在這裡
        // 所以不能寫 using（已在 class 外）
        // 也不能寫 namespace（已在 namespace 裡）
        // 也不能寫 public class（已在 UserModule 裡）

        #endregion
    }
}
```

## Debug 偵錯機制

### 只能 Debug 最後存檔的模組

所有 Server 模組的編譯都透過**同一份 `EEPServer.Core/UserModule.cs`**，每次存檔覆寫。

**流程**：
```
存檔模組 A（"訂單"）
  → UserModule.cs 寫入 A 的程式碼
  → 編譯 → 產出 design/server/{Solution}/訂單.Core.dll
  → UserModule.cs 目前內容 = A 的程式碼

存檔模組 B（"庫存"）
  → UserModule.cs 覆寫為 B 的程式碼
  → 編譯 → 產出 design/server/{Solution}/庫存.Core.dll
  → UserModule.cs 目前內容 = B 的程式碼

此時若要 Debug A —— 不行，因為 UserModule.cs 已是 B
要 Debug A 必須：回 EEP 設計介面點 A → 按存檔 → UserModule.cs 才會變回 A
```

### Debug 步驟

1. 開啟 VS / VS Code，載入 `EEPServer.Core` 專案
2. 在 `EEPServer.Core/UserModule.cs` 開啟要 debug 的方法
3. 在 EEP 設計介面存檔那個模組（才能讓 UserModule.cs 同步為該模組）
4. 在 UserModule.cs 設斷點
5. VS Attach to Process → 選 IIS / Kestrel 的 EEP Web process
6. 透過 Web 操作觸發該方法 → 斷點命中

### 多模組輪流 Debug 的痛點

若一個流程牽涉 3 個模組（A 呼叫 B，B 呼叫 C）要排查問題：
- 存檔 A → 只能 debug A 的進入點
- 若想在 B 設斷點 → 要先存檔 B → 這時 A 不能 debug 了
- 一次只能 debug 一個模組

**實務技巧**：若 A 呼叫 B 是透過 `new DataModule(ClientInfo, "B").CallMethod(...)`，那 B 在自己的 DLL（`B.Core.dll`）中，debug B 就得：
1. 存檔 B（UserModule.cs = B 的程式碼）
2. Attach debugger → 然後從外部觸發 A（A 會 call 到 B）
3. 斷點只能停在 B（UserModule.cs 對應 B）

要同時 debug A+B 是**做不到**的（公版機制）。解決方式只能：把 A 改成「直接把 B 的邏輯 inline 進去」或用 log 觀察 B。

## 在 VS Code 寫 C# 的流程

### 官方沒有直接整合

EEP 設計介面是 Web 版，按存檔才會觸發 `MSBuild.Build`。**VS Code 編輯 `UserModule.cs` 存檔不會更新到 EEP 的 `.cs` 檔**（`.cs` 檔是「使用者程式碼」單獨存在 `design/server/{Solution}/{id}.cs`，UserModule.cs 只是暫存包裝）。

### 實際工作流

1. 在 VS Code 撰寫程式碼（可以開 `EEPServer.Core/UserModule.cs` 享受 IntelliSense、錯誤提示）
2. **手動複製**你寫的方法區段
3. 貼回 EEP 設計介面的「程式碼設計」框
4. 按存檔 → 觸發建置（回到 UserModule.cs 被覆寫的流程）

### 小技巧：以目前 UserModule.cs 為起點

若要從 EEP 取回目前最後存檔的程式碼，可以：
- 直接讀 `EEPServer.Core/UserModule.cs` 的 `#region EEP.Core generated code` ... `#endregion` 之間
- 或直接讀 `design/server/{Solution}/{id}.cs`（使用者寫的原始內容，不含框架包裝）

實際上 `MSBuild.GetCode(projName)` 就是讀 UserModule.cs 的 region 內容（L164-185）。

## 加入新的 using 引用

### 如果程式碼要用「不在自動注入清單」的 namespace

18 個預設注入的 namespace 外（例如 `System.Xml` / `System.Drawing` / `MailKit` / `Dapper` 等），寫程式碼時**不能直接寫 `using`**（會語法錯誤）。

### 解法 1：用完整命名空間（暫時用）

```csharp
public string GetXml()
{
    var doc = new System.Xml.XmlDocument();
    doc.LoadXml("<root/>");
    return doc.OuterXml;
}
```

缺點：每次使用該型別都要打全名。

### 解法 2：修改 `MSBuild.cs` 的 `AppendUsing`（永久方案）

> 檔案：`EEPGlobal.Core/MSBuild.cs` L131-150

編輯 `CreateCodeFile()` 裡的 namespace 清單，加入要引用的：

```csharp
code.AppendUsing(new string[] {
    "EEPBase.Core",
    "EEPBase.Core.Utility",
    "EEPServerTools.Core",
    "EEPServerTools.Core.Components",
    "Newtonsoft.Json.Linq",
    "System",
    "System.Collections.Generic",
    "System.Data",
    "System.Linq",
    "System.Net.Http",
    "System.Threading.Tasks",
    "System.IO",
    "System.Net",
    "System.Text",
    "System.Text.Json",
    "System.Web",
    "NPOI.SS.UserModel",
    "NPOI.XSSF.UserModel",
    "System.Xml",             // ← 新增
    "Dapper"                  // ← 新增（若專案參考此 NuGet）
});
```

改完後要：
1. **重 build `EEPGlobal.Core`**（因為 MSBuild.cs 屬於這個專案）
2. 重啟 EEP Web
3. 之後所有 Server 模組存檔時的包裝就會含新的 using

### 解法 2 的注意事項

- **EEP 升級會覆蓋** — `MSBuild.cs` 是公版檔案，升級 EEP 版本時會被覆寫；要維護升級檢查清單記錄這個修改
- **需要 DLL 參照** — 加 using 只是 import，型別所屬的 DLL 必須在 `EEPServer.Core.csproj` 的參照清單中。若是 NuGet 套件，要到 csproj 加 `PackageReference`
- **只加 `EEPServer.Core.csproj` 不夠**（⚠️ 常見陷阱）— 見下方「加 NuGet 套件的完整流程」

## 加 NuGet 套件的完整流程（⚠️ 常見陷阱）

> 來源：[#474441](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474441)（新樓 slh / Roland 回覆）

### 陷阱情境（真實案例）

客戶 slh 加入 `System.Data.OleDb` NuGet 套件：

1. ✅ 在 `EEPServer.Core.csproj` 加 `<PackageReference Include="System.Data.OleDb" Version="6.0.0" />`
2. ✅ 在 `MSBuild.cs` 的 `AppendUsing` 加 `"System.Data.OleDb"`
3. ✅ 重 build EEPCore，無報錯
4. ✅ 在 EEP 設計介面寫 Server code 用 OleDb，**存檔成功編譯成 `.dll`**
5. ❌ 前端 RWD 呼叫該 Server Method → **Runtime 報錯找不到 `System.Data.OleDb`**

### 根本原因

EEP 有**兩個獨立的 .NET 專案參考鏈**：

```
▓ 編譯鏈（用來產出使用者的 {id}.Core.dll）
  EEPServer.Core  ←  這是使用者程式碼被塞進去的專案
    └→ EEPServerTools.Core
    └→ 其他 NuGet

▓ 執行鏈（Web 應用程式啟動載入的 DLL）
  EEPWebClient.Core  ←  Web 啟動的主程式
    └→ EEPGlobal.Core
    └→ EEPServerTools.Core
    └→ 其他 NuGet

  ⚠ EEPServer.Core 不在執行鏈裡
    EEPWebClient.Core 沒有 reference EEPServer.Core
```

**結果**：
- 加 NuGet 到 `EEPServer.Core` → **只影響編譯鏈**，NuGet 的 DLL 出現在 `EEPServer.Core/bin/...`，**不會** copy 到 Web 的 `bin/Debug/net8.0/`
- 使用者程式碼 `{id}.Core.dll` 編譯時能找到型別（編譯鏈有）→ 存檔成功
- Runtime 時，Web 在 `EEPWebClient.Core/bin/...` 找相依 DLL → **找不到** `System.Data.OleDb.dll` → 報錯

### 正確做法：三個層次都要處理

| 層次 | 要動的地方 | 目的 |
|------|-----------|------|
| **1. 編譯時編譯器認得型別** | `EEPServer.Core.csproj` 加 `<PackageReference>` | 讓 `UserModule.cs` 編譯時不報「找不到型別」錯誤 |
| **2. Runtime 載入 DLL** | **Web 執行鏈中任一專案**的 csproj 加 `<PackageReference>`（以下擇一即可） | 讓 NuGet 的 DLL flow 到 Web 的 bin 目錄 |
|  | - `EEPWebClient.Core.csproj`（最直接） |  |
|  | - `EEPGlobal.Core.csproj` |  |
|  | - `EEPServerTools.Core.csproj` |  |
| **3. 寫程式不用打全名**（可選） | `MSBuild.cs` 的 `AppendUsing` 加 namespace | 寫 `new OleDbConnection(...)` 而不是 `new System.Data.OleDb.OleDbConnection(...)` |

### 三個層次的職責邊界

| 層次 | 缺這個會怎樣 |
|------|-------------|
| 只缺 1（沒加 `EEPServer.Core.csproj`） | 存檔時編譯失敗：`找不到型別 OleDbConnection` |
| 只缺 2（只加 EEPServer.Core） | 存檔成功，Runtime 拋 `FileNotFoundException` — **就是 #474441 的情境** |
| 只缺 3（沒加 `AppendUsing`） | 仍可用，但程式要寫完整命名空間 `System.Data.OleDb.OleDbConnection` |

### 客戶實測

[#474441](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=474441) Roland 建議加 3 個專案：
1. `EEPWebClient.Core.csproj`
2. `EEPGlobal.Core.csproj`
3. `EEPServerTools.Core.csproj`

但客戶實測結果：**只加 2 與 3（`EEPGlobal` + `EEPServerTools`）就能運作**，沒加 `EEPWebClient.Core` 也 OK。

**為什麼**？因為 `EEPWebClient.Core` 在它的參考鏈中已包含 `EEPGlobal` 與 `EEPServerTools`，所以任一加 NuGet 都會 flow 到 `EEPWebClient.Core/bin/`。實務上**加一個就夠**（挑最邏輯相關的，例如資料庫相關加到 `EEPServerTools.Core`）。

### 完整範例：加入 `System.Data.OleDb`

**步驟 1** — `EEPServer.Core/EEPServer.Core.csproj`：
```xml
<ItemGroup>
  <PackageReference Include="System.Data.OleDb" Version="6.0.0" />
  <!-- 其他既有的 NuGet... -->
</ItemGroup>
```

**步驟 2** — `EEPServerTools.Core/EEPServerTools.Core.csproj`（或 `EEPGlobal.Core.csproj`）：
```xml
<ItemGroup>
  <PackageReference Include="System.Data.OleDb" Version="6.0.0" />
  <!-- 其他既有的 NuGet... -->
</ItemGroup>
```

**步驟 3（可選）** — `EEPGlobal.Core/MSBuild.cs` L131-150：
```csharp
code.AppendUsing(new string[] {
    // ... 既有 18 個 ...
    "System.Data.OleDb",      // ← 新增
});
```

**步驟 4** — 重 build：
```bash
cd EEPServer.Core && dotnet build
cd ../EEPServerTools.Core && dotnet build
cd ../EEPGlobal.Core && dotnet build    # 若有改 MSBuild.cs
cd ../EEPWebClient.Core && dotnet build
```

**步驟 5** — **重啟 Web**（IIS Recycle App Pool 或重跑 Kestrel），讓新 DLL 被載入。

### 驗證

在 Server code 裡寫：
```csharp
public object TestOleDb()
{
    using (var conn = new System.Data.OleDb.OleDbConnection("Provider=...;"))
    {
        conn.Open();
        return "OleDb OK";
    }
}
```

前端按鈕 → 呼叫 Server Method：
- ✅ 不報錯 → 三個層次都對
- ❌ 存檔時報 `找不到型別` → 缺步驟 1
- ❌ Runtime 報 `FileNotFoundException` → 缺步驟 2（最常見）
- ❌ 報 `namespace 不存在` → 步驟 3 沒加但又用了短名稱（改寫全名或回頭加 AppendUsing）

### 升級 checklist

升級 EEP 版本時，**多個 csproj 與 MSBuild.cs 都可能被覆寫**，修改記錄要包括：

- [ ] `EEPServer.Core.csproj` 是否保留新增的 `<PackageReference>`
- [ ] `EEPServerTools.Core.csproj`（或 `EEPGlobal.Core.csproj`）是否保留
- [ ] `MSBuild.cs` 的 `AppendUsing` 清單是否保留
- [ ] 重 build 所有專案後 Web 啟動目錄 `bin/Debug/net8.0/` 是否有預期的 NuGet DLL
- [ ] 測試含該 NuGet 的 ServerMethod 能否被前端 RWD 呼叫成功

### 解法 3：新增客製 Provider（進階）

若你的外部整合會跨多個模組，與其每次都依賴 `using`，不如寫一個 `EEPGlobal.Core/Provider/MyIntegrationProvider.cs` 的客製 Provider 類，封裝常用方法，每個模組只要 `new MyIntegrationProvider(Context).DoXxx()`。這樣：
- 不需要修改 `MSBuild.cs`
- 外部 DLL 參照都加在 EEPGlobal.Core 或你的自訂 Core
- 多個模組共用

## 相關 API：`CopyAndBuild`（進階）

另有一個 `MSBuild.CopyAndBuild(projName, outputPath, codes)` 方法（L28-50），會**複製 `EEPServer.Core` 整個資料夾為新的 `{projName}.Core`**，然後在該新資料夾編譯。這樣每個模組有**獨立的 `UserModule.cs`**，不會互相覆寫。

### 用途

目前只在「匯出專案」功能使用（`ExportProj` L2126-2132）。日常存檔流程**不使用**，因為每次複製整個資料夾成本太高。

### 理論上的應用

若要改成「每個模組獨立可 debug」，可以把 `SaveMenu` 從 `MSBuild.Build` 改為 `MSBuild.CopyAndBuild`（已在 MenuProvider.cs L1019 有註解過的候選程式碼），代價是存檔慢很多。目前官方採用共用 `UserModule.cs` 是權衡「存檔速度 vs. debug 便利」的選擇。

## 常見陷阱

| 陷阱 | 說明 | 對策 |
|------|------|------|
| **寫了 `using` 存檔失敗** | 使用者程式碼被包在 class 內，`using` 只能在 namespace 外 | 直接寫 method 即可；框架已注入 18 個 namespace |
| **寫了 `namespace` / `class`** | 同上，語法錯誤 | 只寫 method |
| **只能 debug 最後存檔的模組** | 所有模組共用 UserModule.cs | 要 debug 哪個就回 EEP 按存檔；一次只能 debug 一個 |
| **在 VS Code 存檔不會進 EEP** | UserModule.cs 不是 EEP 認可的程式來源 | 手動複製回 EEP 設計介面按存檔 |
| **改 MSBuild.cs 的 AppendUsing 未重 build** | 只改 .cs 不 build → 存檔還是用舊的包裝 | 重 build EEPGlobal.Core 並重啟 Web |
| **升級 EEP 後 AppendUsing 被還原** | MSBuild.cs 是公版 | 升級 checklist 記錄；或改用客製 Provider 封裝 |
| **編譯錯誤訊息不完整** | `MSBuild.Build` 只抓含 " error " 的行（L95），其他上下文捨棄 | 直接到 `EEPServer.Core/` 下手動 `dotnet msbuild` 看完整輸出 |
| **加 NuGet 只加 EEPServer.Core 存檔 OK 但 Runtime FileNotFoundException** | Web 執行鏈不含 EEPServer.Core | 同時在 EEPGlobal.Core 或 EEPServerTools.Core 加 PackageReference（見「加 NuGet 套件的完整流程」） |
| **多使用者同時存檔衝突** | `lock(typeof(MSBuild))` 只 lock 單一 process，跨機器不保護 | Web 集群部署時，同時存檔同一模組仍可能互相覆寫 UserModule.cs |

## 參考檔案

- `EEPGlobal.Core/MSBuild.cs` — 編譯核心（240 行）
- `EEPGlobal.Core/Provider/MenuProvider.cs` L1006-1037, L2126-2132 — 存檔觸發點、ExportProj 用 CopyAndBuild
- `EEPGlobal.Core/Provider/MenuProvider.cs` L1063-1089 — `BuildAll()`（重建所有模組，升級時用）
- `EEPServer.Core/UserModule.cs` — 執行期動態被覆寫的包裝檔
- `design/server/{Solution}/{id}.cs` — 使用者真實程式碼（純 method，不含包裝）
- `design/server/{Solution}/{id}.Core.dll` — 編譯產出，Runtime 載入
