---
title: Unit Skill Data (RCG_UnitSkillData)
description: A "skill" learned by a character — like equipment but doesn't take a slot; provides passive effects, leader skills, and on-acquire bonuses
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unit Skill Data

> Class name: `RCG_UnitSkillData`

## Purpose

**Template for a skill learned by a character**. Like equipment but doesn't take an equipment slot. Each skill can:
*   Provide passive effects (triggered on various battle triggers)
*   Be marked as a **leader skill** (only effective when the character is in the lead position)
*   Trigger one-time events on learn (e.g., permanent +1 HP, unlock extra draw slot)

Inherits from `RCG_Asset<RCG_UnitSkillData>`. Implements: `RCGI_Status` (battle status system) / `RCGI_Unloackable` (unlock).

## Editor Layout

```
RCG_UnitSkillData: <ID>
    Name(localized)
    Icon                          ← skill icon
    Effects                       ← effects triggered in battle (OnPlay / OnTurnStart / ...)
    AcquireSkillEvents            ← one-time events triggered on learn
    SkillTags                     ← required classes (character must match to learn)
    Tags                          ← general tags
    UnitSkillDescription / Template ← description mode (Auto / Manually)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Skill display name (localized) |
| **Icon** | yes | Skill icon |
| **Effects** | no | Effects to run on various triggers (OnPlay / OnTurnStart / OnAttack...) |
| **AcquireSkillEvents** | no | Events triggered on learning the skill (e.g., expand hand size, permanent enhancement) |
| **SkillTags** | no | Required classes — character's classes must **fully include** these to learn (AND relation) |
| **Tags** | no | General item/skill tags (for categorization, conditional checks) |
| **CanLearnRepeatedly** | — | Can be learned repeatedly (stacking effects) |
| **HideThisSkill** | — | After learning, **doesn't appear in skill panel** (suitable for "trigger acquire bonus only" hidden boosts) |
| **IsLeaderSkill** | — | Whether it's a **leader skill**: only triggers Effects when the character is in the lead position |
| **CanDrop** | — | Whether it can be obtained from drop pools (false = obtainable only via specific paths) |
| **InitCounters** | no | Initial values for counter-type effects |
| **Unlock** | no | Unlock conditions |
| **UnitSkillDescriptionType** | yes | `Auto` (auto-composed from Effects) or `Manually` (hand-written) |
| **UnitSkillDescription** | when Manually | Hand-written description (localized) |
| **DescriptionTemplate** | when Manually | Template string with placeholders like `{(OnPlay.0)}` |

## Behavior

### Trigger Logic
`OnUnitState(triggerOn, data)` for each trigger:
1. Pulls effects from `Effects` for that trigger.
2. If `IsLeaderSkill = true` and the owner **is not** the party leader (compared to `RCG_BattleField.LeadUnit` ID), skips.
3. Otherwise, fires effects in order.

### On Acquire (`OnAquireSkill`)
All `AcquireSkillEvents` are added to `RCG_MapEventManager`, applied to that character (fires immediately or at appropriate timing).

### Description Generation
*   **Auto**: chains "leader skill tag → AcquireEvents description → Effects description".
*   **Manually**: uses `UnitSkillDescription` as base; replaces placeholders like `(OnPlay.0)` via `GetDescriptionParams()`. The `Generate Description Template` button auto-produces a template from current Effects.

### Tooltip Infos
The `Infos` property aggregates all effect `CardInfoData`; leader skills also prepend the `BattleTag_LeadAbility` tag explanation.

## Caveats

*   **`SkillTags` is AND**: listing multiple classes means "must have all of them" to learn; for "any of these", use multiple skills.
*   **`HideThisSkill = true`** suits skills that grant **permanent effect on learn** without runtime passives; players don't see a buff icon but the effect is in place.
*   **`IsLeaderSkill`** ties to the leader switching mechanic; switching leaders does NOT auto re-trigger Effects (`OnPlay` won't fire again).
*   **`CanDrop = false`** skills won't be picked by DropPool, often used for story-unlocked skills.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_UnitSkillData.cs`
*   **Inherits**: `RCG_Asset<RCG_UnitSkillData>`
*   **Implements**: `RCGI_Status` / `RCGI_Unloackable`
*   **AssetGroup**: `EditCharacter`
*   **Default Icon path**: `RCG_SpriteData.SpriteFolder + "/UnitSkills"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Localize Key |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | `Name` |
| `m_Icon` | Icon | `RCG_SpriteData` | `Icon` |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | `Effects` |
| `m_AcquireSkillEvents` | AcquireSkillEvents | `List<RCG_MapEvent>` | `AcquireSkillEvents` |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | `SkillTags` |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | `Tags` |
| `m_CanLearnRepeatedly` | CanLearnRepeatedly | `bool` | `CanLearnRepeatedly` |
| `m_HideThisSkill` | HideThisSkill | `bool` | `HideThisSkill` (Header `HideThisSkillDes`) |
| `m_IsLeaderSkill` | IsLeaderSkill | `bool` | `IsLeaderSkill` |
| `m_CanDrop` | CanDrop | `bool` | `CanDrop` |
| `m_InitCounters` | InitCounters | `List<int>` | `InitCounters` |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | `Unlock` |
| `m_UnitSkillDescriptionType` | DescriptionType | `CardDescriptionType` | — |
| `m_UnitSkillDescription` | Description | `RCG_LocalizeData` | `Conditional(Manually)` |
| `m_DescriptionTemplate` | Template | `string` | `Conditional(Manually)` |

### A.3 Key Methods

*   **`OnUnitState(RCG_EffectTriggerOn, TriggerEffectData)`** — main trigger entry; includes leader check.
*   **`TriggerOnUnitState(RCG_EffectTriggerOn)`** — quick check for trigger system.
*   **`OnAquireSkill(RCG_CharacterData)`** — adds `AcquireSkillEvents` to `RCG_MapEventManager`.
*   **`CheckRequireSkill(HashSet<RCG_SkillTagGenData>)`** — whether character's current skills include all `m_SkillTags`.
*   **`Description` (property)** — Auto / Manually description logic.
*   **`GetDescriptionTemplate / GetDescriptionParams`** — template generation / parameter extraction for Manually mode.
*   **`Status` (property)** — returns `new RCG_StatusGenData(StatusType.UnitSkill, ID)`, implementing `RCGI_Status`.

### A.4 System Interactions

*   **`RCG_CommonEffect`** — trigger effect unit.
*   **`RCG_MapEvent` / `RCG_MapEventManager`** — acquire bonus events system.
*   **`RCG_BattleField.LeadUnit`** — leader skill check.
*   **`RCG_BattleTag.Util.GetData("BattleTag_LeadAbility")`** — leader skill tag.
