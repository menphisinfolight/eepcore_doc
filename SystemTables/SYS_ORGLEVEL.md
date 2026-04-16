# SYS_ORGLEVEL

## 用途

**組織層級定義表**（Organization Level Definition）。

SYS_ORGLEVEL 定義組織架構中的層級體系，從基層主管到總經理。SYS_ORG.LEVEL_NO 關聯此表，流程簽核時用於判斷需要哪個層級的主管簽核。

### 使用場景

| 場景 | 說明 |
|------|------|
| **流程主管層級判斷** | `FlowProvider.GetManagerID()` 查詢 SYS_ORG 的 LEVEL_NO，與目標 levelNo 比較決定是否需要繼續向上查 |
| **組織層級管理** | `orgLevel` infocommand 提供層級定義的 CRUD |

### Provider / 程式碼位置

| 檔案 | 角色 |
|------|------|
| `EEPServerTools.Core/Adapter/Flow.cs` | `GetManagerID()`：比對 LEVEL_NO 決定是否為目標層級的主管，不符則遞迴向上查 |
| `EEPWebClient.Core/design/server/SystemTable.json` | `orgLevel` infocommand：`SELECT * FROM SYS_ORGLEVEL ORDER BY LEVEL_NO` |

---

## 欄位結構

| 欄位名 | 資料類型 | 可為 NULL | 說明 |
|--------|----------|-----------|------|
| **LEVEL_NO** | `nvarchar(6)` PK | NOT NULL | 層級代碼 |
| **LEVEL_DESC** | `nvarchar(40)` | NOT NULL | 層級描述 |

### 跨資料庫差異

各資料庫定義一致，僅 Oracle 使用 `varchar2`。

---

## 主鍵

```
PRIMARY KEY (LEVEL_NO)
```

---

## 預設資料

```sql
INSERT INTO SYS_ORGLEVEL(LEVEL_NO, LEVEL_DESC) VALUES ('0', N'直屬主管');
INSERT INTO SYS_ORGLEVEL(LEVEL_NO, LEVEL_DESC) VALUES ('1', N'主任/課長/副理');
INSERT INTO SYS_ORGLEVEL(LEVEL_NO, LEVEL_DESC) VALUES ('2', N'經理');
INSERT INTO SYS_ORGLEVEL(LEVEL_NO, LEVEL_DESC) VALUES ('3', N'副總');
INSERT INTO SYS_ORGLEVEL(LEVEL_NO, LEVEL_DESC) VALUES ('9', N'總經理');
```

---

## GetManagerID 層級判斷邏輯

```
FlowProvider.GetManagerID(roleID, levelNo, orgKind):
1. 查詢 SYS_ORG 取得 ORG_MAN 和 LEVEL_NO
2. 若 levelNo 為空 或 '0' → 回傳當前 ORG_MAN（直屬主管）
3. 若 levelNo == 當前 LEVEL_NO → 回傳 ORG_MAN（找到目標層級）
4. 若不符 → 以 UPPER_ORG 遞迴向上查（GetManagerID(roleID, upperOrg, levelNo, orgKind)）
5. 若到頂仍未找到 → 回傳最後一個 ORG_MAN
```

---

## 備註

- LEVEL_NO 為字串型別，比對使用 `string.Compare`，'0' 代表直屬主管（不需向上查詢）。
- 預設的 '9'（總經理）通常為組織樹根節點的 LEVEL_NO。
- 層級代碼不需連續，可自由定義（如跳過 4~8）。
