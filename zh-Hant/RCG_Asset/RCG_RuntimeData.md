---
title: 執行時資料 (RCG_RuntimeData) 說明
description: 通用 runtime 變數容器：依 RuntimeStructData 定義動態結構（Int/Bool/Struct/List/Dic/Enum 等）
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 執行時資料

> 程式類別名稱：`RCG_RuntimeData`

## 用途

**通用的 runtime 變數容器**。系統內各處需要「動態結構的儲存格」就用這個——例如戰鬥階段狀態、卡牌觸發時機計數、各種臨時 flag。實際結構由 `RCG_RuntimeStructData`（型別表）定義；`RCG_RuntimeData` 只是**這個結構的一個實例**，含初始值。

繼承自 `RCG_Asset<RCG_RuntimeData>`，實作介面：`UCLI_FieldOnGUI`。

## 編輯器中的樣貌

```
RCG_RuntimeData: <ID>
    StructType   ← 引用的 RCG_RuntimeStructData ID（型別定義）
    DefaultValue ← 此實例的初始值（依 StructType 動態繪製欄位）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **StructType** | 是 | 引用的 `RCG_RuntimeStructData` ID（決定本實例的資料形狀） |
| **DefaultValue** | 是 | 此實例的初始值；型別變動時會自動重建 |

## 行為說明

### Type 變更時自動重建
`DefaultValue` 的 getter 比對 `m_DefaultValue.ID` 與 `m_StructType.ID`，不一致 → null 並重建。**避免舊型別資料殘留**。

### 序列化
`SerializeToJson` 會把 base + `DefaultValue` 一起輸出（key = `DefaultValueKey = "DefaultValue"`）；反序列化時若有此 key 才覆寫 DefaultValue。

### 預設取得 (`InGameData`)
`RCG_RuntimeData.InGameData` 取 `RCG_RuntimeGenData.InGameDataID` 對應的 Asset，作為「全局 In-Game 變數儲存」。

### `RCG_RuntimeDataConst`（檔內子類）
跳過 m_StructType 編輯，直接顯示 DefaultValue。標 `[UCL_IgnoreAsset]` 不會作為獨立 Asset 出現。

## 注意事項

*   **`StructType` 必須先存在**：要先建好對應的 `RCG_RuntimeStructData` 才能在這裡引用。
*   **改 StructType 會清空現有 DefaultValue**：型別不一致直接重建，舊資料會失。
*   **`DefaultType` enum 列舉常用 ID**：`BattleState` / `CardTriggerTiming` 是內建常用 struct type 的快捷方式。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RuntimeData/RCG_RuntimeData.cs`
*   **繼承自**：`RCG_Asset<RCG_RuntimeData>`
*   **實作介面**：`UCLI_FieldOnGUI`
*   **AssetGroup**：`Runtime`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_StructType` | StructType | `RCG_RuntimeStructGenData` | 型別表引用 |
| `m_DefaultValue` (private) | DefaultValue | `RCG_RuntimeObject` | 透過 property 動態建立 |

### A.3 重要 Method 摘要

*   **`InGameData` (static)** — `Util.GetData(InGameDataID)`，全局 in-game 變數儲存。
*   **`DefaultValue` (property)** — 含 type-mismatch 自動重建邏輯。
*   **`SerializeToJson / DeserializeFromJson`** — 含 DefaultValue 子物件序列化。
*   **`GetEnumIDs()`** — 對 Enum 型別取所有 enum 字串值。
*   **`OnGUIDefaultValue(field, dataDic)`** — 繪製 DefaultValue 編輯欄。
*   **`OnGUI(field, dataDic, params)`** — `UCLI_FieldOnGUI` 介面實作。

### A.4 與其他系統的互動

*   **`RCG_RuntimeStructData`** — 型別定義來源。
*   **`RCG_RuntimeObject`** — 動態值的容器類別。
*   **`RCG_RuntimeGenData`** — Asset Entry。
*   **`RCG_RuntimeDataConst`**（同檔）— 不可獨立存在的常數變體。

### A.5 已知議題

*   `RCG_RuntimeDataConst` 標 `[UCL_IgnoreAsset]` 但仍能被引用，使用時要避免誤建 Asset。
