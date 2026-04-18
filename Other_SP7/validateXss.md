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

## 相關討論區

- [#482032](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=482032) — 越南文顯示問題，當時的建議是關 validateXss
- [#473697](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473697) — Sysusers 使用者管理關 validateXss（`$.validateScript = v => v` 全域覆寫）
- [#473029](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473029) — ValidateXss 用途說明

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
