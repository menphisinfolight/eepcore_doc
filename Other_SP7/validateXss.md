# ValidateXss 擴充白名單（不關閉下支援日文/越南文等國際字元）

> 主題：如何在不關閉 `ValidateXss` 保護的前提下，允許輸入日文、越南文、韓文、泰文等 Unicode 字元。
> 適用：EEP SP7
> 相關元件：`UpdateComponent`（Server）、`DataGrid`（RWD）、`$.validateScript`（前端）

## 問題背景

EEP 的 XSS 白名單 regex 只允許下列字元範圍：

| 範圍 | 說明 |
|------|------|
| `\u4e00-\u9fa5` | CJK 基本漢字區（中日共用） |
| `\uff00-\uffff` | 全形區（含半形片假名 `U+FF65-FF9F`、全形英數、全形標點） |
| `a-zA-Z0-9` | 半形英數 |
| 半形/全形常見符號 | `~!@#$%^&*()_-+=?:"{}|,./;'\[]` 及對應全形 |

### 常見被擋的國際字元

| 語系 | Unicode 範圍 | 被擋原因 |
|------|-------------|---------|
| **日文平假名** | `\u3040-\u309F`（`あいうえお` / `とうきょう`） | 不在白名單 |
| **日文全形片假名** | `\u30A0-\u30FF`（`アイウエオ` / `タワー`） | 不在白名單（半形片假名 `U+FF65-FF9F` 才有） |
| 越南文帶聲調 | `\u00C0-\u024F`（`đ ă â ê ô ơ ư` 等） | 不在白名單 |
| 韓文 | `\uAC00-\uD7AF` | 不在白名單 |
| 泰文 | `\u0E00-\u0E7F` | 不在白名單 |
| 阿拉伯文 / 西里爾文 | `\u0600+` / `\u0400+` | 不在白名單 |
| 基本 Emoji | `\u2600-\u27BF` | 不在白名單 |

### 驗證位置（兩處都會擋）

1. **前端** `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.validate.js:5`
   ```javascript
   $.validateScript = function(value) { /* 同一個 regex */ }
   ```
2. **後端** `EEPServerTools.Core/Components/UpdateComponent.cs:528`
   ```csharp
   if (ValidateXss && value.Type == JTokenType.String) {
       if (v.Length == 0 || Regex.IsMatch(v, @"^[...]+$")) { } else
           throw new Exception($"validateXss:{v}");
   }
   ```

**兩邊用完全相同的 regex**。只改前端沒用 — 後端會再擋一次。

## 解法

### 步驟 1：前端全域覆寫（`_global.js`）

在方案的 `_global.js` 中覆寫 `$.validateScript`，加入新的字元範圍。路徑：

```
design/bootstrap/{方案名稱}/_global.js
```

```javascript
// 擴充 XSS 白名單：新增平假名與全形片假名
$.validateScript = function (value) {
    if (!value) return '';
    value = String(value).replace(/</g, '&lt;').replace(/>/g, '&gt;');

    if (/^[\u3040-\u309F\u30A0-\u30FF\u4e00-\u9fa5_\uff00-\uffff_a-zA-Z0-9_`~!@#$%^&*()_\-+=?:"{}|,.\/;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：""【】、；''，。、\s]+$/i.test(value)) {
        return value;
    }
    console.error("value: '" + value + "' has some characters not in whitelist");
    return '';
};
```

**優點**：
- 不動框架檔，不需 rebuild
- 升級時不受影響（屬客製資料層）
- 可從設計後台直接維護

**限制**：只影響該方案，若多方案共用要各自放一份（或改放 `wwwroot/css/*.js` 但升級可能被蓋）。

### 步驟 2：後端擴充 regex（必要）

若只改前端，**存檔時後端仍會擋**。必須編輯：

**檔案**：`EEPServerTools.Core/Components/UpdateComponent.cs`
**行數**：528 行 `ValidateValue` 方法

原本：
```csharp
if (v.Length == 0 || Regex.IsMatch(v,
    @"^[\u4e00-\u9fa5_\uff00-\uffff_a-zA-Z0-9_`~!@#$%^&*()_\-+=?:""{ }|,.\/; '\\[\]·~！@#￥%……&*（）——\-+={}|《》？：""""【】、；''''，。、\s]+$"))
{
}
```

改為（加上 `\u3040-\u309F\u30A0-\u30FF`）：
```csharp
if (v.Length == 0 || Regex.IsMatch(v,
    @"^[\u3040-\u309F\u30A0-\u30FF\u4e00-\u9fa5_\uff00-\uffff_a-zA-Z0-9_`~!@#$%^&*()_\-+=?:""{ }|,.\/; '\\[\]·~！@#￥%……&*（）——\-+={}|《》？：""""【】、；''''，。、\s]+$"))
{
}
```

**注意**：
- 需要 **rebuild `EEPServerTools.Core`**
- **升級 EEP 版本會被覆蓋** — 記錄修改位置（可用本文件或 `AGENTS.md`）
- 升級後要自動跑修改檢查

## 完整 regex（支援多國語言版）

若要一次解決所有國際字元問題：

```
^[\u00C0-\u024F\u0E00-\u0E7F\u3040-\u309F\u30A0-\u30FF\u4e00-\u9fa5\uAC00-\uD7AF_\uff00-\uffff_a-zA-Z0-9_`~!@#$%^&*()_\-+=?:"{}|,./;'\\[\]·~！@#￥%……&*（）——\-+={}|《》？：""【】、；''，。、\s]+$
```

包含：

| 範圍 | 涵蓋內容 |
|------|----------|
| `\u00C0-\u024F` | 拉丁擴充 A/B（越南文、法文、西班牙文、德文等帶聲調） |
| `\u0E00-\u0E7F` | 泰文 |
| `\u3040-\u309F` | 平假名 |
| `\u30A0-\u30FF` | 全形片假名 |
| `\u4e00-\u9fa5` | 中日基本漢字（原有） |
| `\uAC00-\uD7AF` | 韓文音節 |
| `\uff00-\uffff` | 全形符號（原有） |

## 不建議的替代做法

### ❌ 全域放行（等於停用保護）

```javascript
$.validateScript = function (value) { return value; };
```

雖然簡單，但**完全停用前端 XSS 保護**。若有使用者在欄位裡輸入 `<script>alert(1)</script>`，該字串會原樣進 DB，後續在任何顯示處（如 DataGrid）渲染時會執行。**等同沒 ValidateXss**。

### ❌ 對特定 UpdateComponent 關 validateXss

```json
{ "type": "updatecomponent", "id": "ucOrder", "validateXss": false }
```

雖然能寫入日文，但同時也開啟了 XSS 風險。此為「使用者明確要求不關閉」情境下的**不可接受選項**。

### ❌ 把日文 Base64 編碼後存入 DB

```javascript
value = btoa(unescape(encodeURIComponent(value)));
```

繞過 XSS 檢查，但：
- 資料庫內不可讀
- 查詢（WHERE NAME LIKE '%東京%'）失效
- 每次顯示都要 decode

## 相關討論區（完整彙整）

討論區從 2022 到 2026 共有 **32 篇**相關紀錄，以下按語系 / 主題分組。

### 日文相關

| ID | 日期 | 主題 | 結論 |
|----|------|------|------|
| [#482568](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482568) | 2026-04-14 | dfMaster / dg 輸入日文被認為不是有效內容 | DataGrid + UpdateComponent 雙邊 validateXss 設 false 後正常 |
| [#467349](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467349) | 2022-08-27 | DataGrid 欄位日文顯示（三井日本料理） | validateXss 設 false 後資料出現，但部分日文字變方塊字 → 字型另外處理 |

### 越南文相關

| ID | 日期 | 主題 | 結論 |
|----|------|------|------|
| [#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032) | 2025-12-11 | Datagrid 越南文無法顯示 | DataGrid + UpdateComponent 雙邊關 |
| [#480899](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=480899) | 2025-07-04 | 越南文存檔問題 | 同上 |
| [#467407](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467407) | 2022-09-04 | refVal 無法顯示越南文 | refVal 也受影響，同樣 pattern |
| [#466361](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=466361) | 2022-04-12 | 越南文顯示問題 | 舊版需要替換 Provider 檔重 build |
| [#466291](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=466291) | 2022-03-31 | Table 越南文存檔錯誤 | 官方提供了 EEPGlobal.Core\Provider 替換檔（早期版本） |
| [#466234](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=466234) | 2022-03-24 | relval 元件無法顯示越南字 | 同一系列 |
| [#466212](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=466212) | 2022-03-23 | DataGrid 元件無法顯示越南字 | 最早的越南文討論 |

### 泰文相關

| ID | 日期 | 主題 |
|----|------|------|
| [#481637](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481637) | 2025-10-15 | 語系選擇中 Other1 是否可以改為泰文 |

### 特殊字元 / XSS 驗證一般問題

| ID | 日期 | 主題 | 關鍵點 |
|----|------|------|--------|
| [#478410](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=478410) | 2024-11-27 | datagrid textarea `error has some characters` | textarea 也受 validateXss 驗證 |
| [#476227](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=476227) | 2024-10-14 | 單據簽核時 ORA-01704 錯誤 | Oracle 長度限制問題，非 validateXss，但常一起出現 |
| [#473748](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473748) | 2024-03-28 | 特殊字元可以正常存檔 | 關 validateXss |
| [#473697](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473697) | 2024-03-22 | Sysusers.cshtml 使用者管理 validateXss:false | **主頁原始碼覆寫 `$.validateScript = v => v`** |
| [#473638](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473638) | 2024-03-15 | 存檔時符號問題 | 特殊符號不在白名單 |
| [#473197](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473197) | 2024-01-11 | XSS 驗證如何調整 | 整體調整方案 |
| [#473091](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473091) | 2023-12-28 | 特殊字無法顯示 | 官方建議參考 ReadMe 說明 |
| [#473029](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473029) | 2023-12-20 | **ValidateXss 用途說明** | 最完整的背景知識，含 ReadMe 連結 |
| [#469605](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469605) | 2023-06-30 | Server 驗證按鈕後報錯 | |
| [#469187](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469187) | 2023-05-17 | 有特殊字元無法存檔 | |
| [#469008](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=469008) | 2023-04-26 | 驗證功能錯誤 `MenuProvider.validateScript not supported` | 該驗證功能後來**被移除**（功能未支援） |
| [#468606](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=468606) | 2023-03-09 | 選單調整順序時出現 validateXss | Menu 排序誤觸 |
| [#467941](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467941) | 2022-11-23 | Excel 上傳特殊字元的問題 | 匯入路徑也會驗 |
| [#467923](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467923) | 2022-11-22 | 存檔時特殊字元出現「不是有效的內容」 | |
| [#467225](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467225) | 2022-08-11 | 輸入其他語言 | 多語系通用問題 |

### 錯誤訊息「不是有效的內容」觸發情境

前端觸發 validateXss 時的**使用者可見提示**就是「不是有效的內容」，下列篇章都是同一個根因但不同觸發場景：

| ID | 日期 | 觸發場景 | 官方回覆 |
|----|------|----------|----------|
| [#482514](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482514) | 2026-03-26 | 特殊符號 `°`（度） | 關 validateXss；**首次提到「可透過 `EEPWebClient.Core/wwwroot/js/infolight` 自行調整白名單」** |
| [#481105](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=481105) | 2025-07-29 | 資料類型為 HTML 存檔 | 關 validateXss |
| [#473913](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473913) | 2024-04-19 | `#AE` 欄位保存 | Server + DataGrid 雙邊關 |
| [#473030](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473030) | 2023-12-20 | 欄位夾雜注音符號（`ㄅㄆㄇ`） | 雙邊關 |
| [#467068](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=467068) | 2022-07-15 | HtmlEditor 輸入 HTML 標籤 | HtmlEditor 本質要存 `<html>` 標籤，不能開 validateXss |
| [#466634](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=466634) | 2022-05-03 | 存檔時直接顯示「不是有效的內容」 | 預設啟用 validateXss 的說明 |

### 歷史脈絡觀察

1. **2022 年**多為越南文客戶（薩摩亞商旺隆、炫佳）集中回報，官方提供**替換 Provider 檔**的解法（舊版機制）
2. **2023 年**轉為一般特殊字元 / 符號 / 注音問題，官方建議改用 `validateXss: false` + ReadMe 文件宣導
3. **2024 年**出現 **全域覆寫 `$.validateScript`** 的方案（#473697），適用於系統公版頁
4. **2025-2026 年**問題仍持續（日文、越南文新客戶），**#482514（2026-03）首次官方提到「自行調整白名單」**（即本文件建議的方向），但仍無官方標準做法或 config 機制

### 建議

本文件提供的「擴充白名單」方案（前後端 regex 同時修改）是目前**最完整、不失去 XSS 保護**的解法。相較於上述討論區多數建議「關 validateXss」，擴充白名單：

- ✅ 保留對 `<script>` 等危險字串的擋功能
- ✅ 不需每個元件個別設定
- ✅ 可針對客戶語系需求自訂（例如某公司只需日文，就只加 `\u3040-\u309F\u30A0-\u30FF`）
- ⚠️ 需 rebuild 一次、並在升級 checklist 記錄

## 驗證方法

修改前後端之後，測試：

1. 輸入純中文（應通過）
2. 輸入平假名 `ひらがな`（原本擋，修改後通過）
3. 輸入 `東京タワー`（原本擋，修改後通過）
4. 輸入 `<script>alert(1)</script>`（**修改後仍應被擋** — 因為 `<` `>` 被 escape，不符合 regex 白名單）
5. 輸入半形英數（通過）

若步驟 4 沒被擋，代表 regex 改錯了（可能漏掉 escape HTML tag 的段落）。

## 歷次升級檢查清單

每次升級 EEP 版本後，檢查：

- [ ] `UpdateComponent.cs:528` 的 regex 是否被公版覆蓋
- [ ] `_global.js` 的 `$.validateScript` 覆寫是否還在（屬客製資料通常不會被蓋）
- [ ] 測試一筆含日文的資料能否正常存檔 / 顯示
