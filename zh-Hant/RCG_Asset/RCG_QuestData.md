---
title: RCG_QuestData 說明
description: <!-- TODO: 一句話功能摘要 -->
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# RCG_QuestData

> 程式類別名稱：`RCG_QuestData`

## 用途

<!-- TODO: 描述這個 Asset 在遊戲裡負責什麼、什麼情境會用、舉 1-2 個範例。 -->

繼承自 `RCG_Asset<RCG_QuestData>`，實作介面：`UCL.Core.UI.UCLI_FieldOnGUI`

## 編輯器中的樣貌

```
<!-- TODO: 描繪此 Asset 在編輯器內的版面 -->
```

## 主要欄位

| 編輯器顯示 | 必填 | 說明 |
|---|---|---|
| **DarkMistIncreaseWhenMove** | — | <!-- TODO: 說明欄位用途 --> |
| **DarkMistAppearance** | — | <!-- TODO: 說明欄位用途 --> |
| **FogUIType** | — | <!-- TODO: 說明欄位用途 --> |
| **CompleteConditions** | — | <!-- TODO: 說明欄位用途 --> |
| **IncompatibleConditions** | — | <!-- TODO: 說明欄位用途 --> |
| **TopMenuState** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestTags** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestLv** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestLength** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestDarkMistChange** | — | <!-- TODO: 說明欄位用途 --> |
| **DarkMistRelatedSettings** | — | <!-- TODO: 說明欄位用途 --> |
| **Difficulty** | — | <!-- TODO: 說明欄位用途 --> |
| **AddDifficultyAfterPass** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestProgress** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestEventIcons** | — | <!-- TODO: 說明欄位用途 --> |
| **CustomizeMapSize** | — | <!-- TODO: 說明欄位用途 --> |
| **MapWidth** | — | <!-- TODO: 說明欄位用途 --> |
| **MapHeight** | — | <!-- TODO: 說明欄位用途 --> |
| **AutoEnterIfOnly** | — | <!-- TODO: 說明欄位用途 --> |
| **Name** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestObjective** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestIcon** | — | <!-- TODO: 說明欄位用途 --> |
| **BGM** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestGoals** | — | <!-- TODO: 說明欄位用途 --> |
| **FieldEffects** | — | <!-- TODO: 說明欄位用途 --> |
| **Tags** | — | <!-- TODO: 說明欄位用途 --> |
| **Map** | — | <!-- TODO: 說明欄位用途 --> |
| **StoryPool** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestBattles** | — | <!-- TODO: 說明欄位用途 --> |
| **CardDropPools** | — | <!-- TODO: 說明欄位用途 --> |
| **ItemDropPools** | — | <!-- TODO: 說明欄位用途 --> |
| **EquipmentDropPools** | — | <!-- TODO: 說明欄位用途 --> |
| **BattleScene** | — | <!-- TODO: 說明欄位用途 --> |
| **DetailSetting** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestType** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestRandomGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestMixRandomData** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestCaveGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestRuinGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **QuestTestEventsGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **SlayTheSpireGenData** | — | <!-- TODO: 說明欄位用途 --> |
| **SlsHorizontalGenData** | — | <!-- TODO: 說明欄位用途 --> |

## 行為說明

<!-- TODO: 戰鬥 / 載入 / 解鎖時的觸發時機與順序。 -->

## 注意事項

<!-- TODO: 常見的設計反模式 / 容易踩到的坑。 -->

---

## 附錄：程式人員參考 (Programmer Reference)

> 此段以下使用程式內部術語，受眾轉為程式人員與 AI agent。前半段內容請優先採信。

### A.1 類別資訊

*   **檔案路徑**：`CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_QuestData.cs`
*   **繼承自**：`RCG_Asset<RCG_QuestData>, UCL.Core.UI.UCLI_FieldOnGUI`
*   **實作介面**：`UCL.Core.UI.UCLI_FieldOnGUI`

### A.2 欄位對照（自動產生，需人工複核）

| 程式欄位 | 編輯器顯示 | 型別 | Localize Key | 備註 |
|---|---|---|---|---|
| `m_DarkMistIncreaseWhenMove` | DarkMistIncreaseWhenMove | `bool` | `DarkMistIncreaseWhenMove` | |
| `m_DarkMistAppearance` | DarkMistAppearance | `bool` | `DarkMistAppearance` | |
| `m_FogUIType` | FogUIType | `FogUIType` | `FogUIType` | |
| `m_CompleteConditions` | CompleteConditions | `List<RCG_QuestCondition>` | `CompleteConditions` | |
| `m_IncompatibleConditions` | IncompatibleConditions | `List<RCG_QuestCondition>` | `IncompatibleConditions` | |
| `m_TopMenuState` | TopMenuState | `RCG.UI.TopMenuState` | `TopMenuState` | |
| `m_QuestTags` | QuestTags | `List<QuestTag>` | `QuestTags` | |
| `m_QuestLv` | QuestLv | `int` | `QuestLv` | |
| `m_QuestLength` | QuestLength | `int` | `QuestLength` | |
| `m_QuestDarkMistChange` | QuestDarkMistChange | `int` | `QuestDarkMistChange` | |
| `m_DarkMistRelatedSettings` | DarkMistRelatedSettings | `DarkMistRelatedSetting` | `DarkMistRelatedSettings` | |
| `m_Difficulty` | Difficulty | `int` | `Difficulty` | UCL.Core.PA.UCL_IntSlider(0, 100) |
| `m_AddDifficultyAfterPass` | AddDifficultyAfterPass | `int` | `AddDifficultyAfterPass` | |
| `m_QuestProgress` | QuestProgress | `int` | `QuestProgress` | |
| `m_QuestEventIcons` | QuestEventIcons | `List<RCG_SpriteData>` | `QuestEventIcons` | |
| `m_CustomizeMapSize` | CustomizeMapSize | `bool` | `CustomizeMapSize` | |
| `m_MapWidth` | MapWidth | `float` | `MapWidth` | UCL.Core.PA.Conditional("m_CustomizeMapSize", false, true) |
| `m_MapHeight` | MapHeight | `float` | `MapHeight` | UCL.Core.PA.Conditional("m_CustomizeMapSize", false, true) |
| `m_AutoEnterIfOnly` | AutoEnterIfOnly | `bool` | `AutoEnterIfOnly` | |
| `m_Name` | Name | `RCG_LocalizeData` | `Name` | |
| `m_QuestObjective` | QuestObjective | `RCG_LocalizeData` | `QuestObjective` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | `QuestIcon` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | `BGM` | |
| `m_QuestGoals` | QuestGoals | `List<RCG_QuestGoalData>` | `QuestGoals` | |
| `m_FieldEffects` | FieldEffects | `List<ToggleableFieldEffectGenData>` | `FieldEffects` | |
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | `Tags` | |
| `m_Map` | Map | `RCG_PrefabResData` | `Map` | |
| `m_StoryPool` | StoryPool | `RCG_StoryDropPoolGenData` | `StoryPool` | |
| `m_QuestBattles` | QuestBattles | `Dictionary<RCG_EnemyTypeTagGenData, RCG_BattleSetDropPoolGenData>` | `QuestBattles` | |
| `m_CardDropPools` | CardDropPools | `Dictionary<RCG_EnemyTypeTagGenData, RCG_CardDropPoolGenData>` | `CardDropPools` | |
| `m_ItemDropPools` | ItemDropPools | `Dictionary<RCG_EnemyTypeTagGenData, RCG_ItemDropPoolGenData>` | `ItemDropPools` | |
| `m_EquipmentDropPools` | EquipmentDropPools | `Dictionary<RCG_EnemyTypeTagGenData, RCG_EquipmentDropPoolGenData>` | `EquipmentDropPools` | |
| `m_BattleScene` | BattleScene | `RCG_BattleSceneGenData` | `BattleScene` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | `DetailSetting` | |
| `m_QuestType` | QuestType | `QuestType` | `QuestType` | |
| `m_QuestRandomGenData` | QuestRandomGenData | `RCG_QuestRandomGenData` | `QuestRandomGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.RandomGen) |
| `m_QuestMixRandomData` | QuestMixRandomData | `RCG_QuestMixRandomData` | `QuestMixRandomData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.MixRandom) |
| `m_QuestCaveGenData` | QuestCaveGenData | `RCG_QuestCaveGenData` | `QuestCaveGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.CaveGen) |
| `m_QuestRuinGenData` | QuestRuinGenData | `RCG_QuestCaveGenData` | `QuestRuinGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.RuinGen) |
| `m_QuestTestEventsGenData` | QuestTestEventsGenData | `RCG_QuestEventsTestGenData` | `QuestTestEventsGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.TestEvents) |
| `m_SlayTheSpireGenData` | SlayTheSpireGenData | `RCG_SlayTheSpireGenData` | `SlayTheSpireGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.SlayTheSpire) |
| `m_SlsHorizontalGenData` | SlsHorizontalGenData | `RCG_SlsHorizontalGenData` | `SlsHorizontalGenData` | UCL.Core.PA.Conditional("m_QuestType", false, QuestType.SlayTheSpireHorizontal) |

### A.3 重要 Method 摘要

<!-- TODO: 補上影響行為的關鍵 method（OnGUI / Preview / 序列化覆寫等）。 -->

### A.4 與其他系統的互動

<!-- TODO: 列出依賴 / 被依賴的類別與系統。 -->

### A.5 已知議題（選填）

<!-- TODO: TODO/FIXME 摘錄、待重構點。 -->
