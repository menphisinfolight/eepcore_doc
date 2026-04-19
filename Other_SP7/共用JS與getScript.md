# 共用 JS 檔的做法 + `$.getScript` 用法

> 主題：讓 EEP 系統內所有 RWD 作業共同引用一份自訂 JS 的正確做法。
> 相關討論：[#473178](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473178)

## 推薦做法：`_global.js` 方案級注入（升級安全）

**路徑**：`design/bootstrap/{方案名}/_global.js`

EEP 框架會自動注入此檔到該方案的**所有 RWD 頁**（登入後），這是專門為了「方案級全域 JS」設計的機制。

| 面向 | 結果 |
|------|------|
| 是否改框架檔 | ❌ 不需要 |
| 升級影響 | ✅ 不會被覆蓋（屬客製資料層） |
| 維護方式 | ✅ 可從設計後台編輯 |
| 作用範圍 | 該方案**所有 RWD 頁面**（登入後） |
| 登入前 Logon 頁 | ❌ 不會載入 |

### 作法 1：共用 code 直接寫進 `_global.js`（code 不大）

```javascript
// design/bootstrap/{方案}/_global.js

// 自訂工具 function（全站共用）
window.EEP_Utils = {
    formatDate: function(d) {
        return d.getFullYear() + '/' + (d.getMonth()+1) + '/' + d.getDate();
    },
    calcTax: function(amount) {
        return Math.round(amount * 0.05);
    }
};

// 或全站統一的 ajax 錯誤處理
$(document).ajaxError(function(event, jqxhr, settings, thrownError) {
    console.error('AJAX error:', settings.url, thrownError);
});

// 啟動時一次性動作
$(document).ready(function() {
    console.log('EEP 方案已載入');
});
```

### 作法 2：動態載入外部 JS 檔（code 很大想分檔）

```javascript
// design/bootstrap/{方案}/_global.js
$.getScript('/design/bootstrap/{方案}/common-utils.js');
$.getScript('/design/bootstrap/{方案}/biz-helpers.js');
```

把 `common-utils.js` / `biz-helpers.js` 放同目錄（`design/bootstrap/{方案}/` 下）即可，同樣屬客製資料不會被升級覆蓋。

## 不推薦的做法

### ❌ 改 `Views/Design/Index.cshtml` 的 `<head>` 加 `<script>`

[#473178](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473178) 官方原始答覆，但有幾個問題：

- 屬**框架檔案，升級會被覆蓋**，要維護升級 checklist
- Publish 環境要 **rebuild**
- `Views/Design/*.cshtml` 是設計介面頁面，不是 RWD Runtime —  動它不見得影響使用者看到的 RWD 頁
- 要 Runtime 生效得改 `Views/Home/*.cshtml` 才對

**只在**以下情況考慮：
- `_global.js` 不適用（例如要影響所有方案 / 登入前 Logon 頁）
- 已接受升級 checklist 的成本

### ❌ 用 Literal 元件放 `<script src="">`

客戶原本在 [#473178](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473178) 提到的方向。問題：

- **每個 RWD 頁都要手動加 Literal** 才會載入
- 沒有「一次引用全頁面生效」的效果
- 只適合「1-2 頁需要特殊 JS」的情境

## 特殊情況：登入前（Logon 頁）也要用

`_global.js` 只在**登入後**載入。登入畫面也要共用 JS 的話：

| 位置 | 升級風險 | 適用 |
|------|---------|------|
| 改 `Views/Home/Logon.cshtml` 加 `<script>` | 高（升級會覆蓋） | 只影響登入頁 |
| 放 `wwwroot/js/` 被 `bootstrap.infolight` 載入 | 高（同上） | 全站（含登入前） |
| 自訂 middleware 注入 | 低（客製檔案） | 進階做法 |

## `$.getScript` 用法詳解

jQuery 內建 method，**動態載入外部 JS 檔並執行**。類似 `<script src="...">` 但可在 runtime 任意時點呼叫。

### 最簡用法

```javascript
$.getScript('/design/bootstrap/KPI/common-utils.js');
```

- 發 AJAX 請求抓該 JS 檔
- 抓回來直接 `eval()` 執行（定義的 function / 變數會在全域）
- **非同步**，回傳 Promise

### 等載入完再做事（callback）

```javascript
$.getScript('/design/bootstrap/KPI/common-utils.js', function() {
    // 載入完成後才執行
    EEP_Utils.formatDate(new Date());   // 此時 EEP_Utils 已存在
});
```

### 穩健的 Promise 寫法

```javascript
$.getScript('/design/bootstrap/KPI/common-utils.js')
    .done(function() {
        EEP_Utils.formatDate(new Date());
    })
    .fail(function() {
        console.error('載入 common-utils.js 失敗');
    });
```

### 同步載入（避免 race condition）

若下一行立刻要用載入進來的函式，非同步會拿不到。改同步：

```javascript
$.ajax({
    url: '/design/bootstrap/KPI/common-utils.js',
    dataType: 'script',
    async: false     // ★ 同步
});
EEP_Utils.formatDate(new Date());   // 此時一定存在
```

> ⚠️ 現代瀏覽器 console 會警告「sync XHR on main thread」；只在啟動初期載 1-2 檔的情境可接受，不要動態資料載入也用。

### 防止重複載入

`$.getScript` 每次都會**真的重新下載 + 重新執行**。若要避免：

```javascript
// 方式 1：自己用旗標
if (!window._commonUtilsLoaded) {
    $.getScript('/design/bootstrap/KPI/common-utils.js', function() {
        window._commonUtilsLoaded = true;
    });
}

// 方式 2：用 cache: true（預設 false 會加 ?_=時間戳 防 cache）
$.ajax({
    url: '/design/bootstrap/KPI/common-utils.js',
    dataType: 'script',
    cache: true    // 瀏覽器 cache 可重用
});
```

### 在 EEP 的典型使用情境

#### 情境 A：共用工具（`_global.js` 啟動時載一次）

```javascript
// design/bootstrap/KPI/_global.js
$.getScript('/design/bootstrap/KPI/common-utils.js');
$.getScript('/design/bootstrap/KPI/biz-helpers.js');
```

#### 情境 B：按需載入（第三方函式庫體積大）

```javascript
// design/bootstrap/KPI/_index.js — 只首頁要用的 charting lib
$(document).ready(function() {
    if (location.pathname.includes('dashboard')) {
        $.getScript('https://cdn.jsdelivr.net/npm/chart.js');
    }
});
```

#### 情境 C：使用者按下按鈕才載（延遲載入）

```javascript
function onClickReport() {
    $.getScript('/design/bootstrap/KPI/report-exporter.js', function() {
        ReportExporter.generate();
    });
}
```

#### 情境 D：有 dependency 的串接

```javascript
// 先載 utility，再載依賴它的 biz module
$.getScript('/design/bootstrap/KPI/common-utils.js')
    .then(function() { return $.getScript('/design/bootstrap/KPI/order-module.js'); })
    .then(function() { return $.getScript('/design/bootstrap/KPI/report-module.js'); })
    .done(function() {
        OrderModule.init();
        ReportModule.init();
    });
```

## `$.getScript` 與 `<script src="">` 的差異

| 面向 | `<script src="">` | `$.getScript()` |
|------|-------------------|-----------------|
| 載入時機 | HTML 解析時 | JS 執行時（任意時點） |
| 阻塞 | 預設會阻塞 HTML 渲染 | 非阻塞 |
| 取得結果 | 不會知道是否載入成功 | 有 callback / Promise |
| 重複載入 | 瀏覽器自動去重 | 每次都真的執行（除非 `cache:true`） |
| 用途 | 頁面初始固定載入 | 動態 / 條件式載入 |

EEP 寫在 `_global.js` 裡的 `$.getScript`，本質上就是「頁面載入 `_global.js` → 再由它動態載其他共用 JS」的**套娃模式**。

## 常見陷阱

| 陷阱 | 說明 | 對策 |
|------|------|------|
| **路徑寫錯** | 相對路徑是相對 Web 根（`http://localhost:4438/`），不是相對當前 HTML | 用 `/design/...` 絕對路徑較穩 |
| **載入順序亂** | 非同步時不保證順序 | 有 dependency 用 `.then()` 串起來 |
| **ES Module 語法報錯** | `$.getScript` 當傳統 script 執行 | 用 `import/export` 寫的模組不適用；改回 IIFE 或全域賦值 |
| **跨域失敗** | CORS 阻擋 | 外部 CDN 要確認有 CORS header；自家域下的檔沒問題 |
| **生產環境頻寬浪費** | 預設 `cache: false` 每次都重抓 | 明確指定 `cache: true` 或用版本號 |
| **重複載造成記憶體膨脹** | 每次 getScript 都在全域重新定義 | 用旗標或 `cache: true` 避免 |
| **失敗靜默** | 沒加 `.fail()` 時載入失敗看不到 | 每次都加 callback 或 `.fail()` |
| **順序誤用 `done`/`then`** | `.then` 可串接多個 `$.getScript`，`.done` 不行 | 需要串接用 `.then`（Promise 標準） |

## 選擇決策樹

```
要讓多個 RWD 頁共用 JS？
├─ 只影響該方案（最常見）
│    └─ 推：_global.js + $.getScript 載多個檔
│
├─ 要影響所有方案
│    ├─ 登入後 → 每個方案的 _global.js 各自寫一份（或共用 symlink）
│    └─ 登入前（含 Logon）→ 改 wwwroot/js/ 或 Logon.cshtml（接受升級風險）
│
├─ 只有 1-2 頁要用
│    └─ 用 Literal 元件放 <script src="">（不用建立全域機制）
│
└─ 需要第三方 CDN（Chart.js / moment.js 等）
     ├─ 全站用 → _global.js 的 $.getScript
     └─ 按需用 → 按鈕 onclick 才 $.getScript（減輕首次載入）
```

## 相關討論區

| ID | 日期 | 主題 |
|----|------|------|
| [#473178](https://www.infolight.com/cloud_andyhome2_bootstrap/DISDT?type=010&id=473178) | 2024-01-09 | 共用 js 檔的作法（官方答覆：改 Index.cshtml，非最佳解） |

## 推薦一句話

**共用 JS 寫在 `design/bootstrap/{方案}/_global.js`，用 `$.getScript` 動態載入其他 JS 檔。不要改 Index.cshtml。**
