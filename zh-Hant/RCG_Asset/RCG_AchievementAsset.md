---
title: 成就資料 (RCG_AchievementAsset) 說明
description: 玩家可解鎖的成就：條件達成後自動解鎖；可串 Steam 成就，可有前置成就
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# 成就資料

> 程式類別名稱：`RCG_AchievementAsset`

## 用途

**玩家可解鎖的成就模板**。條件達成後自動解鎖（並可同步到 Steam 成就）。支援前置成就鏈、職業 / 角色相關標記、自訂描述覆寫。

繼承自 `RCG_Asset<RCG_AchievementAsset>`。

## 編輯器中的樣貌

```
RCG_AchievementAsset: <ID>
    IsClassAchievement / SkillTag             ← 職業相關旗標（true 時顯示對應職業）
    IsCharacterAchievement / Character        ← 角色相關旗標
    OverrideDescription                       ← 覆蓋自動描述
    OrderIndex                                ← 顯示排序
    HasSteamAchievement / SteamAchievement    ← Steam 成就串接
    Conditions                                ← 達成條件清單（AND）
    HasPrerequisiteAchievement / PrerequisiteAchievement  ← 前置成就
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **IsClassAchievement** | — | 是否為「職業成就」（決定是否顯示 SkillTag 欄位） |
| **SkillTag** | IsClassAchievement=true | 對應的職業（`RCG_SkillTagGenData`） |
| **IsCharacterAchievement** | — | 是否為「角色成就」 |
| **Character** | IsCharacterAchievement=true | 對應的角色（`RCG_CharacterGenData`） |
| **OverrideDescription** | 否 | 覆蓋自動產生的描述（多語系，含參數高亮） |
| **OrderIndex** | — | 排序索引 |
| **HasSteamAchievement** | — | 是否串接 Steam 成就 |
| **SteamAchievement** | HasSteamAchievement=true | Steam 成就 ID |
| **Conditions** | 否 | 達成條件（`RCG_Condition` list；**AND** 關係） |
| **HasPrerequisiteAchievement** | — | 是否有前置成就 |
| **PrerequisiteAchievement** | HasPrerequisiteAchievement=true | 必須先達成的成就 |

## 行為說明

### 條件檢查 (`CheckAchievementCondition`)
*   `m_Conditions` 為空 → 回 false（沒條件視為「不達成」，與 GameChallenge 的「沒條件視為通過」相反）。
*   `m_Conditions.CheckConditions_AND` → 所有條件都滿足才回 true。
*   有前置成就且本成就條件已通過 → 遞迴檢查前置條件。

### 解鎖 (`CheckAchievement`)
1. 若有 Steam 成就且**已達成**（`steamAchievement.GetStat() == true`）→ 直接回 true（避免重覆觸發）。
2. 跑 `CheckAchievementCondition`。
3. 達成 + 有 Steam 成就 → `steamAchievement.SetStat(true)` 上報 Steam。

### 描述生成 (`RequirementDes`)
*   `OverrideDescription` 有設 → 用它（含參數高亮 `Term` 顏色 tag）。
*   否則 → 串接 `m_Conditions` 各自的 `GetShortName()`。

## 注意事項

*   **`m_Conditions` 為空回傳 false**：與直覺相反——空條件不會自動達成。**此設計與 GameChallenge 的 `IsEmpty || Unlocked` 不一致**，要小心。
*   **Steam 成就一旦達成不會重置**：本檔不提供「取消成就」流程。
*   **OverrideDescription 含參數高亮**：用 `RCG_Extensions.TagColors.Term` 對參數上色，編輯時要把參數寫成 `{0}` 之類的佔位符（具體規則見 `RCG_LocalizeData.GetName`）。
*   **前置成就鏈的條件檢查**：`CheckAchievementCondition` 會遞迴跑前置的條件——前置條件滿足才能解鎖本成就，但**前置成就本身是否已解鎖（GetStat == true）這層不影響本成就邏輯**。
*   **註解 `// TODO: use as condition QWQ?`** 標示：`m_SkillTag` / `m_Character` 目前只是「分類旗標」，沒當條件用；未來可能會作為自動條件。

---

## 附錄：程式人員參考 (Programmer Reference)

### A.1 類別資訊
*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AchievementAsset.cs`
*   **繼承自**：`RCG_Asset<RCG_AchievementAsset>`
*   **AssetGroup**：`EditGameSetting`

### A.2 欄位對照

| 程式欄位 | 編輯器顯示 | 型別 | 備註 |
|---|---|---|---|
| `m_IsClassAchievement` | IsClassAchievement | `bool` | |
| `m_SkillTag` | SkillTag | `RCG_SkillTagGenData` | `Conditional(IsClassAchievement)` |
| `m_IsCharacterAchievement` | IsCharacterAchievement | `bool` | |
| `m_Character` | Character | `RCG_CharacterGenData` | `Conditional(IsCharacterAchievement)` |
| `m_OverrideDescription` | OverrideDescription | `RCG_LocalizeData` | |
| `m_OrderIndex` | OrderIndex | `int` | |
| `m_HasSteamAchievement` | HasSteamAchievement | `bool` | |
| `m_SteamAchievement` | SteamAchievement | `UCL_SteamAchievementEntry` | `Conditional(HasSteamAchievement)` |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | AND |
| `m_HasPrerequisiteAchievement` | HasPrerequisiteAchievement | `bool` | |
| `m_PrerequisiteAchievement` | PrerequisiteAchievement | `RCG_AchievementEntry` | `Conditional(HasPrerequisiteAchievement)` |

### A.3 重要 Method

*   **`CheckAchievement(triggerEffectData)`** — 入口：Steam 已達成快檢 → 條件 → 前置 → 上報 Steam。
*   **`CheckAchievementCondition(triggerEffectData)`** — 純條件檢查（不上報）；遞迴前置。
*   **`RequirementDes` (property)** — 自動 / 覆寫描述。

### A.4 與其他系統的互動

*   **`UCL_SteamAchievementEntry`** — Steam SDK 串接。
*   **`RCG_Condition`** — 條件元素。
*   **`RCG_AchievementEntry`** — Asset Entry；預設 `CreatorAchievement`。
*   **`RCG_SkillTagGenData / RCG_CharacterGenData`** — 分類用。

### A.5 已知議題

*   `m_SkillTag` / `m_Character` 標 `// TODO: use as condition QWQ?`，目前只是分類，未自動轉成條件。
*   空 `m_Conditions` 回 false 的行為與直覺相反（與 GameChallenge 不一致）。
