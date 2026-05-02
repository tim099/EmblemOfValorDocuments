---
title: Quest Data (RCG_QuestData)
description: Full definition of one quest (small map) — node layout, battle pools, reward pools, goals, dark mist, random generation
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Quest Data

> Class name: `RCG_QuestData`

## Purpose

**Full definition of one "quest" (small map)**. After entering a quest node from the bigmap, the game generates the corresponding small map: node layout, paths between nodes, enemy configurations, reward pools, dark mist rules, quest goals. Quests support multiple generation modes: hand-edited, pure random, mixed random, Slay-the-Spire style, cave, ruin, etc.

Inherits from `RCG_Asset<RCG_QuestData>`. Implements: `UCL.Core.UI.UCLI_FieldOnGUI`.

## Editor Layout

```
RCG_QuestData: <ID>
    Name / QuestObjective / QuestIcon / BGM / Map prefab
    QuestGoals                 ← quest goals (primary / secondary)
    FieldEffects               ← field effects (toggleable)
    StoryPool                  ← event pool for this quest
    QuestBattles               ← default battle pool (per enemy type)
    Card / Item / EquipmentDropPools  ← reward pools (per enemy type)
    BattleScene                ← default battle scene
    QuestType                  ← Default / RandomGen / MixRandom / SlayTheSpire / CaveGen / RuinGen / TestEvents / TestBattles
    [QuestType-specific sub-settings]      ← various RandomGenData
    DetailSetting              ← conditions / tags / difficulty / dark mist / map size / auto-enter, etc.
    MapEditData                ← node edit data (hidden, edit via QuestEditor)
    [button] Open QuestEditor    ← node editor (Editor playing only)
    [button] Export Quest Info   ← export this quest's monster / battle info to Markdown
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name / QuestObjective** | yes | Quest name / objective description (localized) |
| **QuestIcon** | yes | Quest icon (shown on bigmap node) |
| **BGM** | yes | Background music |
| **Map** | yes | Corresponding small map Prefab |
| **QuestGoals** | no | Primary / secondary quest goals (rewards on completion) |
| **FieldEffects** | no | Field effects for this quest (toggleable) |
| **StoryPool** | no | Story event pool for this quest |
| **QuestBattles** | no | Battle pool by enemy type (Normal / Elite / Boss, etc.) |
| **CardDropPools / ItemDropPools / EquipmentDropPools** | no | Reward pools by enemy type; missing → fallback to EnemyType's default pool |
| **BattleScene** | no | Default battle scene (used when not specified) |
| **QuestType** | yes | Map generation method (determines which sub-setting is shown) |
| **DetailSetting** | yes | Details (conditions / tags / difficulty / dark mist settings / map size / `AutoEnterIfOnly`, etc.) |

`DetailSetting` contains:

| Sub-field | Description |
|---|---|
| **CompleteConditions** | Display prerequisite conditions (AND; hide if not met) |
| **IncompatibleConditions** | Mutually exclusive conditions (AND; hide if not met) |
| **TopMenuState / QuestTags / QuestLv** | Top menu state / tag list / difficulty level |
| **QuestLength / QuestDarkMistChange** | Quest length / dark mist change (multiplied by `DifficultyData.m_DarkMistMult`) |
| **DarkMistRelatedSettings** | Dark mist details: rises on movement, mist appearance, auto-distribute node resistance |
| **Difficulty / AddDifficultyAfterPass** | Quest base difficulty / difficulty increment after clear |
| **QuestProgress / QuestEventIcons** | Progress value / event icons |
| **CustomizeMapSize / MapWidth / MapHeight** | Custom map size |
| **AutoEnterIfOnly** | Auto-enter when this is the only quest on bigmap |

## Behavior

### Random Generation (`GetMapEditData`)
Branches by `QuestType`:
*   `MixRandom` → `m_QuestMixRandomData.RandomGen` + `BalanceDistanceIteration`
*   `RandomGen` / `SlayTheSpire` / `SlsHorizontal` / `CaveGen` / `TestEvents` → respective RandomGen
*   `Default` → reads pre-saved `m_MapEditData` directly
*   `RuinGen` / `TestBattles` → **unimplemented** (logs error)

### Quest Goal Auto-Generation (`GenerateQuestGoals` static)
If a quest has no goals, the static method generates them based on the current bigmap stage's reward pools:
*   **Primary goal** (hash-seeded): 3 rotating rewards — equipment / unit skill / card; requirement is "defeat boss" (Boss tag quest) or "defeat N elites".
*   **Secondary goal**: 3 rotating — soul threshold / dark mist level threshold / common enemy defeat count; rewards are gold / soul / item.

### Append Challenge Goals (`AppendGameChallengeGoals` static)
Marks `RCG_DataService.Ins.m_ChallengeGoalDatas` undone challenge goals as `m_IsChallengeGoal` and appends to the quest goals.

### Clear (`QuestComplete`)
`Difficulty += m_DetailSetting.m_AddDifficultyAfterPass`: difficulty permanently rises after clearing this quest.

### Progress (`QuestProgress`)
*   `BigMapType.Loop` (legacy Loop bigmap) → returns `QuestProgressToComplete` directly.
*   Newer (tag-based) → if any of `BigMapData.QuestProgressTags` matches → returns 1; else 0.

### Condition Checks
*   **`IsPrerequirementMet(selected)`** — if already cleared → false; `CompleteConditions` all AND must pass.
*   **`IsQuestAvailable(selected)`** — if already cleared → false; `IncompatibleConditions` all AND must pass.

## Caveats

*   **Switching `QuestType` requires clearing `MapEditData`**: otherwise leftover random data interferes. `RemoveRandomNodes` clears.
*   **`AutoEnterIfOnly` only effective for permanent / starting-blessing scenarios**: bigmap auto-enters when only this quest remains.
*   **Auto goal generation is deterministic**: uses hash of quest IDs for seed — **the same quest combo always generates the same goals**.
*   **`RuinGen` / `TestBattles` unimplemented**: selecting them logs error without generating.
*   **`m_FieldEffects` elements are `ToggleableFieldEffectGenData`**: can be disabled; runtime uses `FieldEffects` property to filter enabled ones.
*   **`Export Quest Info` button**: exports monster configuration Markdown to `Assets/Export/Quest/<ID>.md` for external review of battle setup.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_MapScripts/RCG_QuestData.cs`
*   **Inherits**: `RCG_Asset<RCG_QuestData>`
*   **Implements**: `UCL.Core.UI.UCLI_FieldOnGUI`
*   **AssetGroup**: `EditQuestSetting`
*   **CurQuest** (static) — most recently constructed RCG_QuestData, for external lookup

### A.2 Field Mapping (selected)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` / `m_QuestObjective` | Name / Objective | `RCG_LocalizeData` | |
| `m_QuestIcon` / `m_BGM` / `m_Map` | icon / BGM / map | `RCG_SpriteData` / `RCG_BGMGenData` / `RCG_PrefabResData` | |
| `m_QuestGoals` | QuestGoals | `List<RCG_QuestGoalData>` | |
| `m_FieldEffects` | FieldEffects | `List<ToggleableFieldEffectGenData>` | |
| `m_Tags` | Tags | `List<RCG_EventTagGenData>` | |
| `m_StoryPool` | StoryPool | `RCG_StoryDropPoolGenData` | |
| `m_MapEditData` | MapEditData | `MapEditData` | `[UCL_HideOnGUI]` |
| `m_QuestBattles` | QuestBattles | `Dictionary<RCG_EnemyTypeTagGenData, RCG_BattleSetDropPoolGenData>` | |
| `m_CardDropPools / m_ItemDropPools / m_EquipmentDropPools` | reward pools | `Dictionary<EnemyType, *DropPoolGenData>` | |
| `m_BattleScene` | BattleScene | `RCG_BattleSceneGenData` | |
| `m_QuestType` | QuestType | `QuestType` enum | 8 modes |
| `m_QuestRandomGenData / m_QuestMixRandomData / m_QuestCaveGenData / m_SlayTheSpireGenData / ...` | various GenData | various classes | `Conditional(matching QuestType)` |
| `m_DetailSetting` | DetailSetting | `DetailSetting` | |

### A.3 Key Methods

*   **`GetMapEditData(isLoadMap)`** — main entry; random gen or load saved per QuestType.
*   **`QuestComplete()`** — Difficulty += after clear.
*   **`GenerateQuestGoals` (static)** — auto-generates primary + secondary goals.
*   **`AppendGameChallengeGoals` (static)** — appends challenge goals.
*   **`GetCardDropPool / GetItemDropPool / GetEquipmentDropPool / GetBattleSet`** — fetch pool by enemy type; falls back if missing.
*   **`IsPrerequirementMet / IsQuestAvailable`** — display condition checks.
*   **`QuestProgress` (property)** — two computation methods (legacy / new).
*   **`SaveGame / LoadGame`** — random-generated MapEditData serialization.
*   **`ExportQuestInfoToJson`** — exports monster info Markdown to `Assets/Export/Quest/`.
*   **`OnGUI(field, dataDic, params)`** — custom rendering (PreviewTexture + Export button).

### A.4 System Interactions

*   **`MapEditData`** — node / path container.
*   **`RCG_QuestDropPool`** — random pool of Quests.
*   **`RCG_BigMapData.QuestProgressTags`** — basis for progress.
*   **`RCG_QuestRandomGenData / RCG_QuestMixRandomData / RCG_QuestCaveGenData / RCG_SlayTheSpireGenData / ...`** — generators per QuestType.
*   **`RCG_QuestGoalData / QuestGoalRequirementData / GoalReward*`** — goal / reward system.
*   **`RCG_DataService.Ins.m_ChallengeGoalDatas`** — source of challenge goals.
*   **`UI.RCG_QuestEditorUI`** — node editor.

### A.5 Known Issues

*   `RuinGen` / `TestBattles` unimplemented — selecting just LogErrors.
*   `GenerateQuestGoals` uses hash seed, no random seed reset — same IDs always generate same goals.
*   `OnGUI`'s old graphics rendering code (GL.Begin / GL.LINES) commented out, replaced by PreviewTexture.
*   Multiple `// QWQ` / `// QWQ23` markers indicate pending refactor and legacy migration points.
