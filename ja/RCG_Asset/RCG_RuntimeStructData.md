---
title: Runtime 結構定義 (RCG_RuntimeStructData) 說明
description: RCG_RuntimeData 用的「型別表」：定義 Int/Float/String/Bool/Json/Enum/Struct/List/Dic 結構
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
translation_status: pending-ja
---

> [!WARNING]
> 翻訳待機中 — このファイルは日本語翻訳が必要です。
参考用に zh-Hant 原文を以下に掲載しています。


# Runtime 結構定義

> 程式類別名稱：`RCG_RuntimeStructData`

## 用途

**`RCG_RuntimeData` 使用的「型別定義表」**。系統需要動態定義資料結構時（不靠 C# 的 class 而靠資料），就在這裡建一個 Struct 描述：欄位有哪些、各自是什麼型別。例如「玩家本局狀態」這個 Struct 可以含 `Score: Int / IsBoss: Bool / Inventory: List<Item>`。

繼承自 `RCG_Asset<RCG_RuntimeStructData>`。

## 編輯器中的樣貌

```
RCG_RuntimeStructData: <ID>
    StructType  ▾ Int / Float / String / Bool / Json / Enum / Struct / Dic / List
    Name(多國語言)
    Fields       (StructType=Struct)  ← 各欄位定義 dic：name → struct type ref
    ElementType  (StructType=List/Dic) ← 元素型別 ref
    Enums        (StructType=Enum)     ← enum 值列表
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **StructType** | 是 | 9 種型別：5 種原始型別（Int/Float/String/Bool/Json）+ Enum + Struct + List + Dic |
| **Name** | 否 | 顯示名（多語系） |
| **Fields** | StructType=Struct | 結構欄位 dic：欄位名 → 對另一個 RuntimeStructData 的引用 |
| **ElementType** | StructType=List/Dic | 元素型別引用 |
| **Enums** | StructType=Enum | 列舉值的字串列表 |

## 行為說明

### 預設原始型別自動產生 (`PrimitiveDic`)
5 種原始型別會在第一次存取時懶建：`{ "Int": ..., "Float": ..., "String": ..., "Bool": ..., "Json": ... }`。**它們不需要手動建 Asset**，使用時直接引用 ID `"Int"` / `"Float"` 等即可。

### 泛型字串解析 (`CreateData`)
ID 帶尖括號（如 `"List<Int>"` 或 `"Dic<String,Float>"`）時自動解析：
*   切出 StructType 字串（List / Dic）。
*   切出 element type 字串（取最後一個逗號後）。
*   建立並 cache 到 `GenericDic`。

### 預覽 (`PreviewData`)
依 StructType 用對應 UI 繪製資料；遞迴展開 Struct / List / Dic。

### 編輯 (`DataOnGUI`)
依 StructType 繪製對應的編輯介面（IntField / NumField / TextArea / BoolField / Popup / 巢狀資料展開等）。

### Json 序列化 (`SerializeDataToJson` / `DeserializeDataFromJson`)
全結構支援 JSON 來回轉換；遞迴處理 Struct/List/Dic。

### 資料變動偵測
`DataOnGUI` 用 `iDataDic[PrevDataKey]` 比對當前資料；不一致就清掉 GUI 子 dic（避免 IntField cache 殘留錯誤輸入）。

## 注意事項

*   **`Event` 型別已被註解**（程式內 `// Event,`）：曾規劃但未實作。
*   **原始型別 Asset 不要手動建**：用 `PrimitiveDic` 自動懶建即可，手動建會跟自動產生的衝突。
*   **List / Dic 的泛型語法**：用 `"List<元素型別ID>"` / `"Dic<KeyID,ValueID>"`（其中 Key 目前只支援 String，第一個值會被忽略只讀最後一個）。
*   **遞迴上限 10 層**：`CreateData(layer)` 防無限遞迴；過深的巢狀 struct 會回 null。
*   **修改 Struct 的 Fields**：原本資料不會自動遷移；改 schema 後舊資料的 dictionary 可能保留多餘 key 或缺漏 key。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeStructData.cs`
*   **繼承自**：`RCG_Asset<RCG_RuntimeStructData>`
*   **AssetGroup**：`Runtime`
*   **常數**：`s_PrimitiveTypes` = 5 種；`GenericTypeDic` = List/Dic（dic 的元素型別數）

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_StructType` | StructType | `StructType` enum | 9 種 |
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Fields` | Fields | `Dictionary<string, RCG_RuntimeStructGenData>` | `Conditional(Struct)` |
| `m_ElementType` | ElementType | `RCG_RuntimeStructGenData` | `Conditional(List/Dic)` |
| `m_Enums` | Enums | `List<string>` | `Conditional(Enum)` |

### A.3 重要 Method 摘要

*   **`PrimitiveDic` / `GenericDic` (static)** — 懶建快取。
*   **`GetAllIDs(useCache)`** — 含 primitives + generics + base IDs。
*   **`CreateData(string)` (override)** — 含泛型字串解析。
*   **`CreateData(int layer = 0)` (instance)** — 建初始值（含遞迴上限）。
*   **`DataOnGUI(data, fieldName, dataDic)`** — 主編輯介面，按 StructType 分流。
*   **`PreviewData(data, fieldName)`** — 主預覽。
*   **`SerializeDataToJson / DeserializeDataFromJson`** — JSON 來回。
*   **`GetRuntimeObject(obj, key/int)` / `SetData(obj, key/int, val)` / `GetList / GetDictionary / GetFields / GetFieldInfo`** — 給外部存取資料的 helper。

### A.4 與其他系統的互動

*   **`RCG_RuntimeData`** — 引用此型別的 Asset。
*   **`RCG_RuntimeObject`** — 動態值容器。
*   **`RCG_RuntimeStructGenData`** — Asset Entry 包裝；含 `GenericTypeLeft = '<'` 等常數。

### A.5 已知議題

*   `Event` 型別在 enum 中註解，存在但不會被使用。
*   `IsNone` 只判斷 Struct 類型的 Fields 為空；其他類型不會被視為 None。
*   `CreateData` 的泛型解析僅取**最後一個** type 參數（適用 Dic 但未來若擴展可能改）。