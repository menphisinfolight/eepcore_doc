# SYSAUTONUM

## 用途

**自動編號流水號表**（System Auto Number）。

SYSAUTONUM 儲存各種自動編號規則的當前流水號狀態。EEP Core 的 `AutoNumber` 元件在新增資料時，會查詢此表取得當前號碼、遞增後回寫，實現不重複的自動編號機制。

### 使用場景

| 場景 | 說明 |
|------|------|
| **單據編號** | 訂單單號 `ORD2026001`、`ORD2026002`… |
| **日期流水** | 依年度/月份重置，如 `202604-001`（Fixed='202604'） |
| **UpdateComponent 整合** | `AutoNumber.Updatecomponent` 屬性可綁定 UpdateComponent，在新增時自動產生編號 |
| **InfoTransaction 整合** | 交易元件的 `AutoNumber` 屬性可指定 AutoNumber 元件，在交易新增時自動填入編號 |

### 運作原理

```
1. Init() → 根據 AutoNoID + Fixed 查詢 SYSAUTONUM，取得 CURRNUM
   - 找到記錄 → 從 CURRNUM 繼續
   - 找不到 → 從 StartValue 開始
2. GetValue() → 回傳編號字串 = Fixed + 補零 + 流水號，並遞增
3. GetSqls() → 產生 UPDATE 或 INSERT SQL，將新 CURRNUM 回寫 SYSAUTONUM
```

### 元件屬性（AutoNumber.cs）

| 屬性 | 類型 | 說明 |
|------|------|------|
| `AutoNoID` | string | 對應 SYSAUTONUM.AUTOID |
| `Updatecomponent` | string | 綁定的 UpdateComponent ID |
| `Field` | string | 目標欄位名稱（必填） |
| `GetFixed` | string | Fixed 前綴設定，支援日期格式如 `_yyyyMMdd` |
| `StartValue` | long | 起始值（預設 1） |
| `Step` | int | 遞增步進（預設 1） |
| `NumDig` | int | 流水號位數（預設 3） |
| `OnGetFixed` | string | 自訂事件，可程式化修改 Fixed 字串 |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Components/AutoNumber.cs` | 核心：Init() / GetValue() / GetSqls() |
| `EEPServerTools.Core/Components/UpdateComponent.cs` | 在 Insert 時觸發 AutoNumber 產生編號 |
| `EEPServerTools.Core/Components/InfoTransaction.cs` | 交易元件整合 AutoNumber |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **AUTOID** | `nvarchar(20)` PK | NOT NULL | 自動編號規則識別碼 |
| **FIXED** | `nvarchar(20)` PK | NOT NULL | 固定前綴字串（同一 AUTOID 可依不同 FIXED 維護獨立流水號） |
| **CURRNUM** | `decimal(10,0)` | NULL | 當前流水號（下次取號的基準值） |
| **DESCRIPTION** | `nvarchar(50)` | NULL | 規則描述 |

### 跨資料庫差異

| 欄位 | SQL Server | Oracle | MySQL | DB2 / Informix |
|------|-----------|--------|-------|----------------|
| CURRNUM | `decimal(10,0)` | `number(10)` | `decimal(10,0)` | `DECIMAL(10,0)` |

---

## 主鍵

```
PRIMARY KEY (AUTOID, FIXED)
```

同一規則（AUTOID）可依不同前綴（FIXED）維護獨立流水號。例如：
- `AUTOID=OrderNo, FIXED=2026` → CURRNUM=128
- `AUTOID=OrderNo, FIXED=2025` → CURRNUM=356

---

## 資料生命週期

```
首次取號：AutoNumber.Init() 查無記錄
      → GetSqls() → INSERT INTO SYSAUTONUM (AUTOID, FIXED, CURRNUM) VALUES (@id, @fixed, @num)

後續取號：AutoNumber.Init() 查到記錄
      → GetSqls() → UPDATE SYSAUTONUM SET CURRNUM = @num WHERE AUTOID = @id AND FIXED = @fixed
```

---

## 備註

- CURRNUM 採「先取號後回寫」策略，在 UpdateComponent 儲存時一併執行 UPDATE/INSERT SQL，確保與資料新增在同一交易中完成。
- 若同一 AUTOID + FIXED 首次取號，會 INSERT 新記錄；之後皆 UPDATE。
- Informix 資料庫有特殊的欄位標記處理（`DbHelper.MarkColumn`）。
