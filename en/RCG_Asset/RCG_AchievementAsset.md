---
title: Achievement Asset (RCG_AchievementAsset)
description: Player-unlockable achievements — auto-unlocked on condition met; can chain Steam achievements and prerequisite achievements
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Achievement Asset

> Class name: `RCG_AchievementAsset`

## Purpose

**Template for player-unlockable achievements**. Auto-unlocked when condition is met (and can sync to Steam achievements). Supports prerequisite achievement chains, class / character markers, and override descriptions.

Inherits from `RCG_Asset<RCG_AchievementAsset>`.

## Editor Layout

```
RCG_AchievementAsset: <ID>
    IsClassAchievement / SkillTag             ← class-related flag (when true, shows the class)
    IsCharacterAchievement / Character        ← character-related flag
    OverrideDescription                       ← override the auto description
    OrderIndex                                ← display order
    HasSteamAchievement / SteamAchievement    ← Steam achievement bridge
    Conditions                                ← achievement conditions (AND)
    HasPrerequisiteAchievement / PrerequisiteAchievement  ← prerequisite achievement
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **IsClassAchievement** | — | Whether it's a "class achievement" (determines whether SkillTag is shown) |
| **SkillTag** | when IsClassAchievement=true | Corresponding class (`RCG_SkillTagGenData`) |
| **IsCharacterAchievement** | — | Whether it's a "character achievement" |
| **Character** | when IsCharacterAchievement=true | Corresponding character (`RCG_CharacterGenData`) |
| **OverrideDescription** | no | Override the auto-generated description (localized, with parameter highlights) |
| **OrderIndex** | — | Sort index |
| **HasSteamAchievement** | — | Whether to bridge to a Steam achievement |
| **SteamAchievement** | when HasSteamAchievement=true | Steam achievement ID |
| **Conditions** | no | Achievement conditions (`RCG_Condition` list; **AND**) |
| **HasPrerequisiteAchievement** | — | Whether there's a prerequisite achievement |
| **PrerequisiteAchievement** | when HasPrerequisiteAchievement=true | The achievement that must be met first |

## Behavior

### Condition Check (`CheckAchievementCondition`)
*   `m_Conditions` empty → returns false (no conditions counts as "not achieved", **opposite of GameChallenge**'s "no conditions = pass").
*   `m_Conditions.CheckConditions_AND` → all conditions must hold for true.
*   With prerequisite achievement and own conditions met → recursively checks prerequisite conditions.

### Unlock (`CheckAchievement`)
1. If has Steam achievement and **already unlocked** (`steamAchievement.GetStat() == true`) → returns true (avoid re-trigger).
2. Runs `CheckAchievementCondition`.
3. Met + has Steam achievement → `steamAchievement.SetStat(true)` to report to Steam.

### Description Generation (`RequirementDes`)
*   `OverrideDescription` set → uses it (with parameter highlights via `Term` color tag).
*   Otherwise → concatenates `m_Conditions`'s `GetShortName()`.

## Caveats

*   **Empty `m_Conditions` returns false**: counter-intuitive — empty doesn't auto-pass. **Inconsistent with GameChallenge**'s `IsEmpty || Unlocked` — be careful.
*   **Steam achievement, once unlocked, doesn't reset**: this file doesn't provide a "revoke" flow.
*   **OverrideDescription supports parameter highlight**: uses `RCG_Extensions.TagColors.Term` to color parameters; when editing, write parameters as `{0}` placeholders (see `RCG_LocalizeData.GetName` for exact rules).
*   **Prerequisite achievement chain**: `CheckAchievementCondition` recursively runs the prerequisite's conditions — those must hold for this to unlock, but **whether the prerequisite achievement itself is unlocked (GetStat == true) doesn't affect this achievement's logic**.
*   **`// TODO: use as condition QWQ?` comment**: `m_SkillTag` / `m_Character` are currently just "category flags", not used as conditions; future may auto-promote them.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AchievementAsset.cs`
*   **Inherits**: `RCG_Asset<RCG_AchievementAsset>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
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

### A.3 Key Methods

*   **`CheckAchievement(triggerEffectData)`** — entry: Steam already-unlocked quick-check → conditions → prerequisite → report to Steam.
*   **`CheckAchievementCondition(triggerEffectData)`** — pure condition check (no reporting); recurses prerequisite.
*   **`RequirementDes` (property)** — auto / override description.

### A.4 System Interactions

*   **`UCL_SteamAchievementEntry`** — Steam SDK bridge.
*   **`RCG_Condition`** — condition element.
*   **`RCG_AchievementEntry`** — Asset Entry; default `CreatorAchievement`.
*   **`RCG_SkillTagGenData / RCG_CharacterGenData`** — for categorization.

### A.5 Known Issues

*   `m_SkillTag` / `m_Character` marked `// TODO: use as condition QWQ?` — currently just categorization, not auto-promoted to conditions.
*   Empty `m_Conditions` returns false, counter-intuitive (inconsistent with GameChallenge).
