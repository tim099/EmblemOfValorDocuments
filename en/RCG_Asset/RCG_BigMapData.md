---
title: Big Map Data (RCG_BigMapData)
description: A complete bigmap (chapter) skeleton — quest generation modes, state machine, dark mist, unlock, starting equipment, progress rules
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Big Map Data

> Class name: `RCG_BigMapData`

## Purpose

**The complete chapter skeleton of one bigmap**. Each BigMap is a full game journey: from start to boss, stepping on quest nodes in between, advancing progress by quest tag. This data defines:
*   Quest generation mode (default rules / quest pool / pool+mix)
*   Multi-state machine (`MapState`: each state has its own quest pool, reward pools, transition conditions)
*   Dark mist settings, unlock conditions, starting equipment, available challenge goals
*   Permanent + final quest pools

Inherits from `RCG_Asset<RCG_BigMapData>`. Implements: `RCGI_Unloackable`.

## Editor Layout

```
RCG_BigMapData: <ID>
    Name / Description / QuestIcon / BGM / BigMap (prefab)
    BigMapBackground
    TopMenuState
    QuestGenerateType         ← Default / QuestPool / QuestPoolMix
    MaxShowingQuestCount      ← max quests shown at once (Default mode)
    QuestProgressToComplete   ← quest progress required to unlock final (0 = ignore)
    QuestPoolSetting          ← (QuestPool/QuestPoolMix) multi-state machine
    PermanentQuests           ← (QuestPoolMix) always-shown quests (e.g., rest)
    FinalQuests               ← (QuestPoolMix) final quests (after Final condition met)
    QuestProgressTags         ← Boss quest tags (Forest/Mountain/Boss, etc.)
    DetailSetting             ← hide flags / tutorial / unlock / starting equipment, etc.
```

## Main Fields (selected)

| Editor Display | Required | Description |
|---|---|---|
| **QuestGenerateType** | yes | `Default` (default rules) / `QuestPool` (pure quest pool) / `QuestPoolMix` (pool + permanent + final mixed) |
| **MaxShowingQuestCount** | when Default | Max quests shown at once (default 3) |
| **QuestProgressToComplete** | — | Progress required to unlock final; > 0 enables progress indicator |
| **QuestPoolSetting** | when QuestPool/Mix | Multi-state machine (each `MapState` has its own quest pool / reward pools / transition conditions) |
| **PermanentQuests** | when QuestPoolMix | Always-shown quests (e.g., rest stops): always appear, **don't count toward display limit** |
| **FinalQuests** | when QuestPoolMix | Final quests (appear only after Final condition met) |
| **QuestProgressTags** | when Default/Mix | Tags counted as "progress quests" (usually Boss) |
| **DetailSetting** | yes | Detailed: HideThisBigMap / RandomQuestOrder / Difficulty / EnterBigMapTutorial / ShowMode / IsTutorial / BigMapType (Default/Loop) / Unlock / OverridingCharacters / StartingEquipments / AvailableChallenges, etc. |

`MapState` per-stage data:
*   `MaxShowingQuestCount` / `QuestDropSettings` (conditional weighted quest pool list)
*   `EquipmentRewardPools / UnitSkillRewardPools / CardRewardPools / ItemRewardPools`: stage-specific reward pools
*   `NextStateConditions` (OR) / `FinalConditions` (OR)

## Behavior

### `EnterBigMap(newLoop)`
On a new run, resets progress and cleared-quest record.

### `GenerateQuests()`
Branches by `QuestGenerateType`:
*   **Default**: not handled here, handled by other manager.
*   **QuestPool**: pure draw from current MapState's quest pool.
*   **QuestPoolMix**: complex logic —
    1. Starting blessing event (first entry)
    2. Last quest was permanent / special → generate permanent quests + skip option
    3. Last quest was Boss → re-randomize (filtered by ProgressTag)
    4. Final condition met → show final quests only
    5. Default: dedup environment tags, mix Boss + normal quests by ratio

### `QuestStart(questData)`
On entering a quest, if `CurState.CheckFinal()` is true → flag as final map (triggers victory UI on win).

### `PassedQuest(questId, progress)`
On completing a quest:
1. Add ID to `m_PassedQuests`.
2. Write to `MapStatePassQuestCount` / `PassQuestCount` Tag (GameTag system).
3. Check `CurState.CheckTransition()` → advance to next MapState (and reset MapStatePassQuestCount).

### `QuestCompleteAsync()`
If final map → show `RCG_VictoryUI`, player chooses to continue (continue = next Loop).

## Caveats

*   **`QuestGenerateType.Default`** is the legacy rule; new maps should use `QuestPoolMix`.
*   **`PermanentQuests` doesn't count toward display limit**: use this for always-shown quest nodes like rest stops or permanent shops.
*   **`m_QuestProgressToComplete` and `FinalQuests`** are used together: FinalQuests doesn't appear until progress threshold met.
*   **`HideThisBigMap` true** comes from more than `m_HideThisBigMap`: also "OverrideAvailableGameModes doesn't include current GameVersion" causes hiding. Demo version uses this to lock chapters.
*   **`ShowMode`** controls visibility under tutorial mode: `ShowIfEnableTutorial` / `ShowIfDisableTutorial` toggle on tutorial state.
*   **`Tutorial` hardcoded ID**: `BigMap_Challenge_Card`, the tutorial-specific bigmap.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_BigMapData.cs`
*   **Inherits**: `RCG_Asset<RCG_BigMapData>`
*   **Implements**: `RCGI_Unloackable`
*   **AssetGroup**: `EditQuestSetting`
*   **Constants**: `TutorialName = "BigMap_Challenge_Card"`

### A.2 Field Mapping (selected)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` / `m_Description` | Name / Description | `RCG_LocalizeData` | |
| `m_QuestIcon` | QuestIcon | `RCG_SpriteData` | |
| `m_BGM` | BGM | `RCG_BGMGenData` | |
| `m_BigMap` | BigMap | `RCG_PrefabResData` | bigmap prefab |
| `m_BigMapBackground` | BigMapBackground | `RCG_SpriteData` | |
| `m_TopMenuState` | TopMenuState | `TopMenuState` enum | |
| `m_QuestGenerateType` | QuestGenerateType | `QuestGenerateType` enum | |
| `m_MaxShowingQuestCount` | MaxShowingQuestCount | `int` | `Conditional(Default)` |
| `m_QuestProgressToComplete` | QuestProgressToComplete | `int` | |
| `m_QuestPoolSetting` | QuestPoolSetting | `QuestPoolSetting` (nested) | `Conditional(QuestPool / QuestPoolMix)` |
| `m_PermanentQuests` / `m_FinalQuests` | permanent / final | `MapState` | `Conditional(QuestPoolMix)` |
| `m_QuestProgressTags` | QuestProgressTags | `List<RCG_QuestData.QuestTag>` | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` (nested) | |

### A.3 Key Methods

*   **`EnterBigMap(newLoop)`** — entry to bigmap; resets progress on new run.
*   **`GenerateQuests()`** — main quest generation entry.
*   **`GenerateQuestsQuestPool()`** / **`GenerateQuestsQuestPoolMix()`** — two implementations.
*   **`QuestStart(questData)`** — on quest start; flags final map.
*   **`PassedQuest(questId, progress)`** — on completion; updates tags, advances MapState.
*   **`QuestCompleteAsync(token, map)`** — on quest complete; final map shows victory UI.
*   **`AutoTriggerQuests()`** — starting blessing / auto-enter single-quest scenarios.
*   **`CurState`** — `m_QuestPoolSetting.m_MapStates[saveData.m_State]`.
*   **`HideThisBigMap`** / **`ShowThisBigMap`** — multi-factor visibility check.
*   **`MapState.GetQuestDropPool`** — picks pool by conditional weights.
*   **`MapState.CheckTransition` / `CheckFinal`** — state transition / final check (OR).

### A.4 System Interactions

*   **`RCG_BigMapManager`** — runtime main entry.
*   **`RCG_QuestData`** / **`RCG_QuestDropPool`** — quest system.
*   **`RCG_BigMapGenData`** — Asset Entry; default `BigMap_Challenge_Card`.
*   **`BigMapSaveData`** — runtime progress storage.
*   **`RCG_DifficultyData.m_AdditionalQuestProgress`** — difficulty-bonus progress.
*   **`RCG_GameSettingData.m_GameVersion`** — Demo lock check.
*   **`RCG_TutorialService`** — tutorial-mode visibility check.

### A.5 Known Issues

*   `// QWQ23` markers throughout, indicating legacy data format migration and pending verifications.
*   `GenerateQuestsQuestPoolMix` is a very long method (300+ lines), with many commented-out LogErrors — hard to maintain.
