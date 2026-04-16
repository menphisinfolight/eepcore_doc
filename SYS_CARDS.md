# SYS_CARDS

## 用途

**首頁卡片定義表**（Dashboard Card Definition）。

SYS_CARDS 定義首頁儀表板的卡片元件，包含卡片類型、標題、屬性、尺寸、圖示、顏色、排序等。使用者可依需求選擇要顯示的卡片。

### 主要使用場景

| 場景 | 說明 |
|------|------|
| **儀表板客製化** | 定義首頁顯示的各類資訊卡片 |
| **資料視覺化** | 圖表、輪播、行事曆等視覺化元件 |
| **待辦整合** | 流程待辦、通知、歷史記錄卡片 |
| **多方案支援** | 不同方案可設定不同的預設卡片組合 |
| **權限控管** | 透過 USERCARDS/GROUPCARDS 控制可見性 |

---

## 欄位結構

| 欄位名 | 資料類型 | 說明 |
|--------|----------|------|
| **CARDID** | `int IDENTITY(1,1)` PK | 卡片識別碼（自動遞增） |
| **CARDTYPE** | `nvarchar(20)` | 卡片類型 |
| **CAPTION** | `nvarchar(30)` | 卡片標題 |
| **ATTRIB** | `nvarchar(1024)` | 卡片屬性（JSON 格式的設定） |
| **SIZE** | `int` | 卡片尺寸（寬度佔比） |
| **HEIGHT** | `int` | 卡片高度 |
| **ICON** | `varchar(30)` | 圖示名稱 |
| **COLOR** | `varchar(30)` | 顏色設定 |
| **SEQ_NO** | `varchar(4)` | 排序序號 |
| **ITEMTYPE** | `nvarchar(20)` | 所屬方案（預設 `SOLUTION1`） |

---

## 主鍵

```
PRIMARY KEY (CARDID)
```

---

## 卡片類型（CARDTYPE）

| 類型 | 說明 | ATTRIB 屬性 |
|------|------|-------------|
| **todo** | 流程待辦事項 | - |
| **history** | 流程歷史記錄 | - |
| **notify** | 流程通知 | - |
| **slide** | 圖片輪播 | TABLE, CHARTCOLUMNS, TEXTCOLUMN, IMAGEFOLDER, conditions |
| **chart** | 圖表 | TABLE, XCOLUMN, YCOLUMN, CHARTTYPE, conditions |
| **calendar** | 行事曆 | TABLE, TITLECOLUMN, TEXTCOLUMN, STARTDATECOLUMN, ENDDATECOLUMN, conditions |
| **excel** | 資料表格 | TABLE, COLUMNS, SORT, isForm, conditions |
| **url** | 外部連結 | URL 字串 |
| **page** | 內部頁面 | 頁面路徑 |

---

## ATTRIB JSON 屬性說明

### 通用屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| **TABLE** | `string` | 資料來源表格名稱 |
| **conditions** | `array` | 篩選條件（FIELD_NAME, CONDITION, CONTENT, ANDOR） |

### slide（輪播）屬性

| 屬性 | 說明 |
|------|------|
| **CHARTCOLUMNS** | 圖片欄位名稱 |
| **TEXTCOLUMN** | 標題欄位名稱 |
| **IMAGEFOLDER** | 圖片資料夾（預設 Images） |

### chart（圖表）屬性

| 屬性 | 說明 |
|------|------|
| **XCOLUMN** | X 軸欄位（分類） |
| **YCOLUMN** | Y 軸欄位（數值），可逗號分隔多個 |
| **CHARTTYPE** | 圖表類型（Line, Bar, Pie 等） |

### calendar（行事曆）屬性

| 屬性 | 說明 |
|------|------|
| **TITLECOLUMN** | 事件標題欄位 |
| **TEXTCOLUMN** | 事件內容欄位 |
| **STARTDATECOLUMN** | 開始日期欄位 |
| **ENDDATECOLUMN** | 結束日期欄位 |

### excel（資料表格）屬性

| 屬性 | 說明 |
|------|------|
| **COLUMNS** | 顯示欄位（逗號分隔） |
| **SORT** | 排序設定 |
| **isForm** | 是否表單模式（影響表格樣式） |

---

## 關聯表

### USERCARDS（使用者卡片設定）

| 欄位 | 說明 |
|------|------|
| **USERID** | 使用者帳號 |
| **CARDID** | 卡片識別碼（關聯 SYS_CARDS） |

### GROUPCARDS（群組卡片設定）

| 欄位 | 說明 |
|------|------|
| **GROUPID** | 群組識別碼（`00` 為預設群組） |
| **CARDID** | 卡片識別碼（關聯 SYS_CARDS） |

---

## 核心程式碼位置

| 檔案 | 角色 |
|------|------|
| `SystemTable.Core/UserModule.cs` | 卡片資料操作邏輯 |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.systeminfocard.js` | 前端卡片渲染 |
| `EEPGlobal.Core/Provider/SecurityProvider.cs` | 卡片權限查詢 |

---

## 卡片尺寸與高度

| SIZE 值 | 說明 |
|---------|------|
| `0` | 標準寬度 |
| `>0` | 大寬度（2 倍標準寬度 + 間距） |

| HEIGHT 值 | 說明 |
|-----------|------|
| 數值 | 卡片高度（px） |

---

## 卡片渲染流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ SYS_CARDS   │────▶│ 讀取卡片    │────▶│ 解析 ATTRIB │
│ 定義卡片    │     │ 設定        │     │ JSON        │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌─────────────┐           ▼
                    │  顯示卡片   │◀────┌─────────────┐
                    │  在儀表板   │     │ 根據類型    │
                    └─────────────┘     │ 渲染內容    │
                                        └─────────────┘
```

---

## 新增卡片自動處理

根據 `SystemTable.Core/UserModule.cs`：

```csharp
public bool ucSYS_CARDS_onAfterApplied(object sender, dynamic data, List<string> sqls)
{
    if (data.inserted.Count > 0)
    {
        // 取得新卡片 ID
        var cardID = result.Rows[0][0];
        // 自動加入預設群組（GROUPID='00'）
        newSqls.Add("DELETE FROM GROUPCARDS WHERE CARDID = " + cardID);
        newSqls.Add("INSERT INTO GROUPCARDS (GROUPID, CARDID) VALUES ('00', " + cardID + ")");
    }
}
```

**說明**：新增卡片時自動加入 `GROUPCARDS` 的預設群組（`00`），讓所有使用者都能看到新卡片。

---

## 備註

- COLOR、SEQ_NO、SIZE、ITEMTYPE 為後期擴充欄位。
- ITEMTYPE 預設值為 `SOLUTION1`，支援多方案不同卡片配置。
- 與 USERCARDS / GROUPCARDS 搭配控制使用者/群組可見的卡片。
- ATTRIB 欄位儲存 JSON 格式，依 CARDTYPE 不同有不同的屬性結構。
- 流程相關卡片（todo/history/notify）透過 `$.fn.flow.renderFlowCard()` 渲染。
- 建議定期檢查 ATTRIB JSON 格式的有效性，避免前端渲染錯誤。
