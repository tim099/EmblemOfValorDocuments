---
title: Difficulty Data (RCG_DifficultyData)
description: Global multipliers for a difficulty level — HP / atk / shop price / dark mist / field effects / prerequisite difficulty
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Difficulty Data

> Class name: `RCG_DifficultyData`

## Purpose

**Template for a difficulty level** — Easy / Normal / Hard / Nightmare / Tutorial all use different IDs of `RCG_DifficultyData`. Defines for that difficulty:
*   HP / attack / shop price multipliers
*   Enemy base skill level bonus
*   Starting resources
*   Camp fire decay rate
*   Item usage limit
*   Persistently applied field effects
*   Prerequisite for unlocking this difficulty (which difficulties must be cleared)

Inherits from `RCG_Asset<RCG_DifficultyData>`.

## Editor Layout

```
RCG_DifficultyData: <ID>
    Name / IconSprite / IsHidden / SortOrder
    BaseLevel              ← monster base level (then run through UnitLevelData curve)
    HealthMult / AtkMult / PriceMult / SoulPriceMult / DarkMistMult
    SellPriceMult          ← multiplier for selling items / equipment
    InitResources          ← bonus starting resources
    CampFireDecreaseRate / CampFireHealPercentageMult
    ItemUsageLimit         ← item usage cap per battle
    AdditionalLength / AdditionalQuestProgress
    EnemySkillLevel        ← base enemy skill level
    FieldEffects           ← persistent field effects
    RequiredCompletedDifficulty  ← prerequisite (OR: any one cleared unlocks this)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name / IconSprite** | yes | Display name and icon |
| **IsHidden** | — | Hide from difficulty menu (test use) |
| **SortOrder** | yes | Menu sort order |
| **BaseLevel** | yes | Monster base level (then transformed through `RCG_UnitLevelData` curve) |
| **HealthMult / AtkMult** | yes | HP / attack multiplier (applied on top of `UnitLevelData.GetMaxHP / GetAtkMult` results) |
| **PriceMult / SoulPriceMult** | yes | Shop / soul shop price multipliers |
| **DarkMistMult** | yes | Dark mist accumulation rate multiplier |
| **SellPriceMult** | — | Multiplier for player selling items (default 0.5) |
| **InitResources** | no | Bonus starting resources |
| **CampFireDecreaseRate** | — | Camp fire usage decay rate (per-use deduction percentage) |
| **CampFireHealPercentageMult** | — | Camp fire heal percentage multiplier |
| **ItemUsageLimit** | — | Per-battle item use cap (0 = unlimited) |
| **AdditionalLength** | — | Bigmap length bonus |
| **AdditionalQuestProgress** | — | Quest progress bonus |
| **EnemySkillLevel** | — | Enemy base skill level bonus (affects `MonsterLevelActionData.GetAction` level) |
| **FieldEffects** | no | Field effects always applied |
| **RequiredCompletedDifficulty** | no | Prerequisite difficulty (**OR**: any one cleared unlocks) |

## Behavior

### Where Multipliers Apply
These mult values are applied at various points:
*   `UnitLevelData.GetMaxHP / GetAtkMult` apply `HealthMult / AtkMult`.
*   Shop UI applies `PriceMult / SoulPriceMult`.
*   Sell price calculation applies `SellPriceMult`.
*   Dark mist accumulation logic applies `DarkMistMult`.
*   `MonsterLevelActionData.GetAction` adds `EnemySkillLevel`.

### `OnVictory()`
Called on victory + chosen "continue"; auto `++m_EnemySkillLevel` — for "infinite mode", each clear loop raises enemy skill level by one.

### Description
`LocalizedDescription` fetches i18n key `<ID>_Description`, so **the description must be in zh-Hant.txt / en.txt**, not edited directly on the Asset.

## Caveats

*   **`RequiredCompletedDifficulty` is OR**: any one satisfied unlocks. List all acceptable prerequisites for multi-tier unlocking.
*   **`OnVictory` permanently mutates the Asset**: `m_EnemySkillLevel` is a serialized field — winning persists. **This means choosing the same difficulty next time is harder** — this is the "infinite mode" design, not a bug.
*   **`Difficulty_Tutorial`** (`TutorialDifficultyID`) is the tutorial-specific ID, mandatorily used at game start.
*   **`m_FieldEffects` always applied**: applied on every battle start; can't be removed.
*   **`Description` uses i18n key**: editing the description field on the Asset has no effect; must add to language files.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DifficultyData.cs`
*   **Inherits**: `RCG_Asset<RCG_DifficultyData>`
*   **AssetGroup**: `EditGameSetting`
*   **Default ID**: `Difficulty_Normal` (also constructor default)

### A.2 Field Mapping (selected)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_SpriteData` | |
| `m_IsHidden` | IsHidden | `bool` | |
| `m_SortOrder` | SortOrder | `int` | Default 1 |
| `m_BaseLevel` | BaseLevel | `int` | Default 1 |
| `m_FieldEffects` | FieldEffects | `List<RCG_FieldEffectGenData>` | |
| `m_HealthMult` / `m_AtkMult` / `m_PriceMult` / `m_SoulPriceMult` / `m_DarkMistMult` | various Mults | `float` | Default 1f |
| `m_SellPriceMult` | SellPriceMult | `float` | Default 0.5f |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_CampFireDecreaseRate` / `m_CampFireHealPercentageMult` | camp fire | `float` | |
| `m_ItemUsageLimit` | ItemUsageLimit | `int` | |
| `m_AdditionalLength` / `m_AdditionalQuestProgress` | progress bonus | `float / int` | |
| `m_EnemySkillLevel` | EnemySkillLevel | `int` | |
| `m_RequiredCompletedDifficulty` | RequiredCompletedDifficulty | `List<RCG_DifficultyGenData>` | OR relation |

### A.3 Key Methods

*   **`OnVictory()`** — `++m_EnemySkillLevel` (infinite mode increment).
*   **`LocalizedName / LocalizedDescription`** — i18n; description goes through `<ID>_Description` key.
*   Constructor defaults `ID = "Difficulty_Normal"`.

### A.4 System Interactions

*   **`RCG_UnitLevelData.GetMaxHP / GetAtkMult`** — applies HealthMult / AtkMult.
*   **`RCG_MonsterLevelActionData.GetAction`** — adds EnemySkillLevel.
*   **`RCG_DataService.Ins.m_DifficultyData`** — runtime application.
*   **`RCG_DifficultyGenData`** — Asset Entry; `TutorialDifficulty` is the tutorial ID.

### A.5 Known Issues

*   `OnVictory` permanently increments `EnemySkillLevel` — saves in Asset; players see accumulated level bonuses across saves on the same difficulty (**this is the intended "infinite challenge" design**).
