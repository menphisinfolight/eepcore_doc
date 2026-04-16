# TransformType (Adapter)

| 項目 | 說明 |
|------|------|
| 檔案路徑 | `EEPGlobal.Core/Adapter/TransformType.cs` |
| 行數 | 758 行 |
| 命名空間 | `EEPGlobal.Core.Adapter` |

## 用途

欄位型別智慧辨識引擎，根據欄位中文名稱、參數格式及自訂字典（jieba 分詞字典），自動判定欄位應使用的 EEP 欄位型別（K/KA/KN/T/N/D/DT/DE/A/R/RV/C/S/O/F/P/E 等）。

## 主要方法

### GetColumnType(columnName, tableNames)

對單一欄位名稱進行型別辨識，使用正則規則：

| 規則 | 型別 |
|------|------|
| 年月日/年月 | DE（日期編輯） |
| 時間 | DT |
| 日期/日$/期限$ | D |
| 額$/量$/值$/價$/稅$ 等 | N |
| 照片/圖片/圖像 | P |
| 檔案/文件/附件 | F |
| 名稱$ | KN |
| 項目/序號$ | KA |
| ID/號$ | K |
| 內容/說明/備註 | A |

### GetType(data, tableNames, type)

批次處理 MasterList / DetailList 等多組欄位，type 可為 `word`、`excel`、`edit_excel`。

### ChangeType（靜態方法）

核心辨識邏輯，處理流程：
1. 清理參數格式（去除 `<>`、tab）
2. 檢查特殊名稱（身分證→TID、郵件→EMAIL、簽名→SG）
3. 分析參數內容：群組→G/GR、樞紐→X、加總→N、日期格式→DE/DT、關聯表→R/RV/KR、選項→S/O/C
4. 查私有字典（userdict_pri.txt）、公有字典（userdict_pub.txt）
5. Fallback 到正則規則

## 字典格式

路徑：`design/jieba/userdict_pub.txt` / `userdict_pri.txt`

每行格式：`欄位名 權重 型別`，例如：`預設 1 T`

## 備註

- 不同 type（word / excel / edit_excel / query_excel）允許的欄位型別不同
- KA 在同一組中只允許出現一次（IsKA 旗標）
- 型別決定後會自動設定 DefaultValue（如 D→$TODAY、N→14,2）和 Size
