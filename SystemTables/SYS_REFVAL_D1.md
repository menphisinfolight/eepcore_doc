# SYS_REFVAL_D1

## 用途

**下拉選單參照明細表**（Reference Value Detail / Dropdown Window Columns）。

SYS_REFVAL_D1 定義下拉選單視窗中顯示的欄位清單。當使用者點擊下拉選單時，彈出的視窗可以顯示多個欄位（而不僅是 DISPLAY_MEMBER），此表定義視窗中要顯示哪些欄位及其表頭和寬度。

### 使用場景

| 場景 | 說明 |
|------|------|
| **多欄位下拉視窗** | 下拉選單彈出視窗顯示多個欄位，方便使用者識別選項 |
| **欄位寬度調整** | 設定各欄位在視窗中的顯示寬度（像素） |
| **表頭自訂** | 設定欄位顯示的表頭文字（可與來源表欄位名不同） |

### 關聯表

```
SYS_REFVAL_D1 ──(REFVAL_NO)──> SYS_REFVAL  （下拉定義，多對一）
```

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPRWDTools.Core/Editors/Refval.cs` | RefVal 元件：讀取 SYS_REFVAL_D1 定義視窗欄位 |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.js` | 前端 RefVal 元件實作，處理下拉視窗欄位顯示 |
| `EEPWebClient.Core/wwwroot/js/infolight/bootstrap.infolight.js` | Bootstrap 版本的 RefVal 元件 |
| `EEPWebClient.Core/wwwroot/js/infolight/jquery.infolight.design.js` | 設計階段的 RefVal 設定 |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **REFVAL_NO** | `varchar(30)` PK | NOT NULL | 參照定義識別碼（對應 SYS_REFVAL.REFVAL_NO） |
| **FIELD_NAME** | `varchar(30)` PK | NOT NULL | 欄位名稱（來源表的欄位名） |
| **HEADER_TEXT** | `varchar(20)` | NULL | 欄位表頭文字（空則顯示 FIELD_NAME） |
| **WIDTH** | `int` | NULL | 欄位寬度（像素，預設 120） |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (REFVAL_NO, FIELD_NAME)
```

---

## 預設欄位邏輯

若 SYS_REFVAL_D1 無記錄，下拉視窗預設顯示：

| 欄位 | 說明 |
|------|------|
| **valueField** | 值欄位（對應 SYS_REFVAL.VALUE_MEMBER） |
| **textField** | 顯示欄位（對應 SYS_REFVAL.DISPLAY_MEMBER） |

寬度預設為 120 像素。

---

## 備註

- 此表是 SYS_REFVAL 的子表，一個下拉定義可有多個顯示欄位。
- FIELD_NAME 必須是來源表中實際存在的欄位名稱。
- 若無 SYS_REFVAL_D1 記錄，下拉視窗預設只顯示 valueField 與 textField 兩個欄位。
