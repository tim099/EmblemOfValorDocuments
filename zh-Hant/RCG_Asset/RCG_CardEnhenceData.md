---
title: 卡牌強化資料 (RCG_CardEnhenceData) 說明
description: 卡牌強化的「分支模板」：強化條件、加 effect、改費用、改卡牌類型、禁用使用類型
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 卡牌強化資料

> 程式類別名稱：`RCG_CardEnhenceData`

## 用途

**卡牌強化的「分支模板」**。每個 Asset 描述「強化能做什麼」：加新 effect、改費用、改卡牌類型、加標籤、改名（如 α、β、γ）。卡牌端 `RCG_CardData.m_EnhencePool` 引用一個 `RCG_CardEnhenceDropPool`，池內掉的就是這些 `RCG_CardEnhenceData` 分支。

繼承自 `RCG_Asset<RCG_CardEnhenceData>`。

## 編輯器中的樣貌

```
RCG_CardEnhenceData: <ID>
    LocalizeName              ← 強化名（α / β / γ；空白用 +）
    Rarity                    ← 強化分支稀有度
    Conditions                ← AND 條件群（哪些卡能套用此強化）
    Effects                   ← 強化加上的新效果
    CardTags                  ← 強化加上的卡牌標籤
    EnhenceSettings           ← 額外的強化設定（修改既有 effect 的條件式套件）
    BannedUsedType            ← 禁止此使用類型的卡套用此強化
    CostAlter                 ← 費用變化（+/-）
    SetCardType / CardType    ← 是否強制改卡牌類型（例：詠唱→普通）
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **LocalizeName** | 否 | 強化命名替代（如 α / β / γ）；空白用 `+` 前綴 |
| **Rarity** | 是 | 強化分支稀有度（影響掉落權重） |
| **Conditions** | 否 | AND 條件式：套用此強化的卡需滿足全部條件 |
| **Effects** | 否 | 強化會加到卡上的新效果 |
| **CardTags** | 否 | 強化加上的卡牌標籤（不重複時才加） |
| **EnhenceSettings** | 否 | 進一步修改既有效果的設定（如：「OnPlay 第一個效果改傷害」） |
| **BannedUsedType** | 否 | 禁止特定使用類型的卡牌套用（例：`Unplayable` 卡禁止強化） |
| **CostAlter** | — | 費用變化值（可正可負） |
| **SetCardType** | — | 是否強制改卡牌類型 |
| **CardType** | SetCardType=true | 改成什麼類型（`Default` / `Chant`） |

## 行為說明

### 條件檢查 (`CheckCondition`)
1. `m_BannedUsedType` 含目標卡的 `UsedType` → 不可用。
2. `m_Conditions.CheckCondition(data)` 不通過 → 不可用。
3. 每個 `EnhenceSettings` 各自 `CheckCondition(data)` 全通過 → 可用。

### 套用 (`Enhence(card)`)
1. 把 `m_LocalizeName` 寫入卡片的 `m_EnhenceLocalize`（顯示用）。
2. 加入所有 enabled 的 `Effects`（深拷貝）。
3. `Cost += m_CostAlter`。
4. `SetCardType` → 套用新類型。
5. `CardTags` → 加標籤（不重複）。
6. 對每個 `EnhenceSettings` 跑 `Enhence(enhenceData)`（修改既有 effect）。

## 注意事項

*   **強化是「永久套用」**：clone 過的卡片會被改名 / 加效果 / 改費用，原始卡不動。
*   **`BannedUsedType` 通常會包含 `Unplayable`**：某些「無法手動打出」的卡（系統卡、隱藏效果卡）不該被強化；曾有自動補上 `Unplayable` 的反序列化邏輯（已註解）。
*   **`EnhenceSettings` 與 `Effects` 的差異**：`Effects` 是「加新的 effect」；`EnhenceSettings` 是「修改現有 effect」（例如：把第一個傷害效果改成 +5 傷害）。
*   **空白 `LocalizeName`** 表示用預設 `+1` / `+2` 顯示；非空白會被 `+α` / `α` 之類取代。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardEnhenceData.cs`
*   **繼承自**：`RCG_Asset<RCG_CardEnhenceData>`
*   **AssetGroup**：`EditItems`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_LocalizeName` | LocalizeName | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Conditions` | Conditions | `RCG_CE_AND_Condition` | AND 群組 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | |
| `m_EnhenceSettings` | EnhenceSettings | `List<RCG_CardEnhenceSetting>` | |
| `m_BannedUsedType` | BannedUsedType | `List<RCG_UsedTypeTagGenData>` | |
| `m_CostAlter` | CostAlter | `int` | |
| `m_SetCardType` | SetCardType | `bool` | |
| `m_CardType` | CardType | `CardType` enum | `Conditional(SetCardType)` |

### A.3 重要 Method 摘要

*   **`CheckCondition(EnhenceData)`** — BannedUsedType + Conditions + EnhenceSettings 三層檢查。
*   **`Enhence(RCG_CardData)`** — 主套用邏輯：寫名 → 加 effects → 改 cost → 改 cardType → 加 cardTags → 套 EnhenceSettings。
*   **`EnhenceSettings` (property)** — 過濾 `IsEnable = true` 的 settings。

### A.4 與其他系統的互動

*   **`RCG_CardData.m_EnhencePool`** / **`GetEnhenceBranchs`** — 引用此資料的入口。
*   **`RCG_CardEnhenceDropPool`** — 強化分支隨機池。
*   **`RCG_CardEnhenceCondition.EnhenceData`** — 條件檢查與套用時的傳遞容器。
*   **`RCG_CardEnhenceSetting`** — 修改既有 effect 的子設定。
*   **`RCG_CardEnhenceGenData`** — Asset Entry 包裝；預設 ID = `"Defense"`。

### A.5 已知議題

*   `DeserializeFromJson` 內有「自動補 Unplayable 到 BannedUsedType」的邏輯被註解，標示舊版自動行為已停用。
