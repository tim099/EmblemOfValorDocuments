---
title: RCG_BigMapData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_BigMapData

> 程式類別名稱：`RCG_BigMapData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_BigMapData>`，實作介面：`RCGI_Unloackable`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **HideThisBigMap** | — | <!-- TODO: 說明欄位用途 --> |
| **OverrideAvailableGameModes** | — | <!-- TODO: 說明欄位用途 --> |
| **RandomQuestOrder** | — | <!-- TODO: 說明欄位用途 --> |
| **ShowMode** | — | <!-- TODO: 說明欄位用途 --> |
| **Difficulty** | — | <!-- TODO: 說明欄位用途 --> |
| **EnterBigMapTutorial** | — | <!-- TODO: 說明欄位用途 --> |
| **SortingOrder** | — | <!-- TODO: 說明欄位用途 --> |
| **ShowTutorialIntro** | — | <!-- TODO: 說明欄位用途 --> |
| **StartBlessingEvent** | — | <!-- TODO: 說明欄位用途 --> |
| **IsTutorial** | — | <!-- TODO: 說明欄位用途 --> |
| **BigMapType** | — | <!-- TODO: 說明欄位用途 --> |
| **MapStatePassQuestCount** | — | <!-- TODO: 說明欄位用途 --> |
| **PassQuestCount** | — | <!-- TODO: 說明欄位用途 --> |
| **OverridingCharacters** | — | <!-- TODO: 說明欄位用途 --> |
| **StartingEquipments** | — | <!-- TODO: 說明欄位用途 --> |
| **Unlock** | — | <!-- TODO: 說明欄位用途 --> |
| **AvailableChallenges** | — | <!-- TODO: 說明欄位用途 --> |
| **Conditions** | — | <!-- TODO: 說明欄位用途 --> |
| **Weight** | — | <!-- TODO: 說明欄位用途 --> |
| **DropPool** | — | <!-- TODO: 說明欄位用途 --> |
| **MaxShowingQuestCount** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestDropSettings** | — | <!-- TODO: 說明欄位用途 --> |
| **EquipmentRewardPools** | — | <!-- TODO: 說明欄位用途 --> |
| **UnitSkillRewardPools** | — | <!-- TODO: 說明欄位用途 --> |
| **CardRewardPools** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemRewardPools** | — | <!-- TODO: 說明欄位用途 --> |
| **NextStateConditions** | — | <!-- TODO: 說明欄位用途 --> |
| **FinalConditions** | — | <!-- TODO: 說明欄位用途 --> |
| **MapStates** | — | <!-- TODO: 說明欄位用途 --> |
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **Description** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestIcon** | — | <!-- TODO: 說明欄位用途 --> |
| **BGM** | — | <!-- TODO: 說明欄位用途 --> |
| **BigMap** | — | <!-- TODO: 說明欄位用途 --> |
| **MaxShowingQuestCount** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestProgressToComplete** | — | <!-- TODO: 說明欄位用途 --> |
| **BigMapBackground** | — | <!-- TODO: 說明欄位用途 --> |
| **TopMenuState** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestGenerateType** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestPoolSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **PermanentQuests** | — | <!-- TODO: 說明欄位用途 --> |
| **FinalQuests** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestProgressTags** | — | <!-- TODO: 說明欄位用途 --> |
| **DetailSetting** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_BigMapData.cs`
*   **繼承自**：`RCG_Asset<RCG_BigMapData>, RCGI_Unloackable`
*   **實作介面**：`RCGI_Unloackable`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_HideThisBigMap` | HideThisBigMap | `bool` | `HideThisBigMap` | |
| `m_OverrideAvailableGameModes` | OverrideAvailableGameModes | `List<RCG_GameSettingData.GameVersion>` | `OverrideAvailableGameModes` | |
| `m_RandomQuestOrder` | RandomQuestOrder | `bool` | `RandomQuestOrder` | |
| `m_ShowMode` | ShowMode | `ShowMode` | `ShowMode` | |
| `m_Difficulty` | Difficulty | `int` | `Difficulty` | UCL.Core.PA.UCL_IntSlider(0, 100) |
| `m_EnterBigMapTutorial` | EnterBigMapTutorial | `TutorialEnum` | `EnterBigMapTutorial` | |
| `m_SortingOrder` | SortingOrder | `int` | `SortingOrder` | |
| `m_ShowTutorialIntro` | ShowTutorialIntro | `bool` | `ShowTutorialIntro` | |
| `m_StartBlessingEvent` | StartBlessingEvent | `bool` | `StartBlessingEvent` | |
| `m_IsTutorial` | IsTutorial | `bool` | `IsTutorial` | |
| `m_BigMapType` | BigMapType | `BigMapType` | `BigMapType` | |
| `m_MapStatePassQuestCount` | MapStatePassQuestCount | `RCG_GameTagEntry` | `MapStatePassQuestCount` | |
| `m_PassQuestCount` | PassQuestCount | `RCG_GameTagEntry` | `PassQuestCount` | |
| `m_OverridingCharacters` | OverridingCharacters | `List<RCG_CharacterGenData>` | `OverridingCharacters` | |
| `m_StartingEquipments` | StartingEquipments | `Dictionary<RCG_CharacterGenData, List<RCG_EquipmentGenData>>` | `StartingEquipments` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` | |
| `m_AvailableChallenges` | AvailableChallenges | `List<RCG_GameChallengeGenData>` | `AvailableChallenges` | |
| `m_Conditions` | Conditions | `List<RCG_Condition>` | `Conditions` | |
| `m_Weight` | Weight | `int` | `Weight` | |
| `m_DropPool` | DropPool | `RCG_QuestDropPoolGenData` | `DropPool` | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `MaxShowingQuestCount` | |
| `m_QuestDropSettings` | QuestDropSettings | `List<QuestDropSetting>` | `QuestDropSettings` | |
| `m_EquipmentRewardPools` | EquipmentRewardPools | `List<RCG_EquipmentDropPoolGenData>` | `EquipmentRewardPools` | |
| `m_UnitSkillRewardPools` | UnitSkillRewardPools | `List<RCG_UnitSkillDropPoolGenData>` | `UnitSkillRewardPools` | |
| `m_CardRewardPools` | CardRewardPools | `List<RCG_CardDropPoolGenData>` | `CardRewardPools` | |
| `m_ItemRewardPools` | ItemRewardPools | `List<RCG_ItemDropPoolGenData>` | `ItemRewardPools` | |
| `m_NextStateConditions` | NextStateConditions | `List<RCG_Condition>` | `NextStateConditions` | |
| `m_FinalConditions` | FinalConditions | `List<RCG_Condition>` | `FinalConditions` | |
| `m_MapStates` | MapStates | `List<MapState>` | `MapStates` | |
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_Description` | Description | `RCG_LocalizeData` | `Description` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | `QuestIcon` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | `BGM` | |
| `m_BigMap` | BigMap | `RCG_PrefabResData` | `BigMap` | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `MaxShowingQuestCount` | UCL.Core.PA.Conditional(nameof(m_QuestGenerateType), false, QuestGenerateType.Default) |
| `m_QuestProgressToComplete` | QuestProgressToComplete | `int` | `QuestProgressToComplete` | |
| `m_BigMapBackground` | BigMapBackground | `RCG_SpriteData` | `BigMapBackground` | |
| `m_TopMenuState` | TopMenuState | `RCG.UI.TopMenuState` | `TopMenuState` | |
| `m_QuestGenerateType` | QuestGenerateType | `QuestGenerateType` | `QuestGenerateType` | |
| `m_QuestPoolSetting` | QuestPoolSetting | `QuestPoolSetting` | `QuestPoolSetting` | UCL.Core.PA.Conditional(nameof(m_QuestGenerateType), false, QuestGenerateType.QuestPool, QuestGenerateType.QuestPoolMix) |
| `m_PermanentQuests` | PermanentQuests | `MapState` | `PermanentQuests` | UCL.Core.PA.Conditional(nameof(m_QuestGenerateType), false, QuestGenerateType.QuestPoolMix) |
| `m_FinalQuests` | FinalQuests | `MapState` | `FinalQuests` | UCL.Core.PA.Conditional(nameof(m_QuestGenerateType), false, QuestGenerateType.QuestPoolMix) |
| `m_QuestProgressTags` | QuestProgressTags | `List<RCG_QuestData.QuestTag>` | `QuestProgressTags` | UCL.Core.PA.Conditional(nameof(m_QuestGenerateType), false, QuestGenerateType.Default, QuestGenerateType.QuestPoolMix) |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | `DetailSetting` | |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
