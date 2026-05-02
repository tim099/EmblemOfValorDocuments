---
title: Unit Data (RCG_UnitData)
description: Master template for every unit on the battlefield (monsters, player characters, summons) — HP, appearance, AI, skills all defined here
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unit Data

> Class name: `RCG_UnitData`

## Purpose

**The master template for every unit on the battlefield** — monsters, player characters, summons are all instances of this class. Includes: HP / combat power, appearance (sprite, Spine, DragonBone, 3D prefab), AI behavior (which skills used per state), initial actions, core unit settings. Replaces the legacy `RCG_MonsterData`.

Inherits from `RCG_Asset<RCG_UnitData>`.

## Editor Layout

```
RCG_UnitData: <ID>
    UnitSetting (m_UnitSetting)        ← core settings: HP, tags, init actions, appearance, positioning
    MonsterStates (m_MonsterStates)    ← AI state machine: actions usable in each state
    Preview (right side)               ← live appearance, skill list, HP
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **UnitSetting** | yes | Core unit settings (HP / tags / init actions / appearance / positioning) |
| **MonsterStates** | yes | AI state machine: dictionary keyed by state ID, value is the list of `MonsterAction` usable in that state. **Must contain a `Default` state** (auto-added by the system) |

`UnitSetting` contains:

| Sub-field | Description |
|---|---|
| **Name** | Display name (localized) |
| **MaxHP** | Max health |
| **CombatEffectiveness** | Combat power metric; auto-set to `MaxHP` on serialize if 0 |
| **MonsterTags** | Monster tags (race, attribute, etc.) — affects certain card / equipment effects |
| **InitActions** | Actions triggered once on entry (buffs/debuffs, summons) |
| **DetailSetting** | Detailed settings (attack count, counters, etc.; varies by type) |
| **SummonSetting** | Settings used when summoned |
| **UnitDisplayerType** | Display mode: Sprite / DragonBone / Spine / 3D Model / Prefab |
| **SpriteDisplayData / DragonBoneDisplayData / ...** | Display data for each renderer (Conditional) |
| **Pos / PosAt** | Position offset, coordinates |
| **BaseLevel / UnitGenData** | Base level and unit ID wrapper |
| **HasAI / Classes / UnitSkills** | AI toggle, classes (for card skill matching), unit skills |

## Behavior

### AI State Machine (MonsterStates)
Each monster has multiple "states" (e.g., `Default`, `Berserk`, `Phase2`); each state has a list of usable `MonsterAction`. At runtime, `MonsterStateTransition` decides when to switch states; the action selection rule decides which action to use that turn.

### Display Modes
*   **Sprite**: Static image (simplest, good for placeholders).
*   **DragonBone / Spine**: 2D skeletal animation.
*   **Model3DDisplayer / Prefab**: 3D model / custom prefab.
The corresponding fields show only after switching `UnitDisplayerType`.

### Init Actions
Auto-applied on entry. Common uses: self-buff, summon followers, set counter initial values.

### Preview (in-editor)
Shows on the right: avatar, max HP, monster tags, init action descriptions, skill icons + names per state. **Low RAM mode** skips avatar rendering to save memory.

## Caveats

*   **The `Default` state must exist**: auto-added by `OnGUI`, but set it up to be safe.
*   **`CombatEffectiveness` fallback**: if 0 on `SerializeToJson`, it's overwritten with `MaxHP`. If you want to preserve 0, special handling is needed.
*   **`NullMonsterID = "Null"`, `DefaultUnitID = "Devil"`, `BattleSceneMonsterID = "BattleScene"`**: these are reserved system IDs — don't use them for regular monsters.
*   **Skill / Action / SummonSetting in the editor** reference data from `RCG_MonsterActionData` / `RCG_UnitSkillData`; this file only holds ID wrappers.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_Battles/RCG_Monsters/RCG_UnitData.cs`
*   **Inherits**: `RCG_Asset<RCG_UnitData>`
*   **AssetGroup**: `EditBattleSetting`, sort = `RCG_UnitData`

### A.2 Field Mapping (outer)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_UnitSetting` | UnitSetting | `RCG_UnitSetting` (nested) | Main core data |
| `m_MonsterStates` | MonsterStates | `Dictionary<string, MonsterState>` | AI state machine; key is state ID |

`UnitSetting` contains `m_Name` / `m_MaxHP` / `m_CombatEffectiveness` / `m_MonsterTags` / `m_InitActions` / `m_DetailSetting` / `m_SummonSetting` / `m_UnitDisplayerType` + corresponding DisplayData / `m_Pos` / `m_PosAt` / `m_BaseLevel` / `m_UnitGenData` / `m_HasAI` / `m_Classes` / `m_UnitSkills`.

### A.3 Key Methods

*   **`Preview` / `OnGUI`** — editor rendering; OnGUI auto-adds the `Default` MonsterState.
*   **`SerializeToJson`** — fills the `m_CombatEffectiveness == 0` fallback.
*   **`Avatar` / `GetAvatar(RCG_BattleUnit)`** — renderer image accessor.
*   **`CreateSelectAssetPage`** — opens `RCG_UnitDataEditorPage`.
*   **`LocalizeName`** — `GetData().LocalizedName`.

### A.4 System Interactions

*   **`RCG_BattleUnit`** — runtime battle unit instance.
*   **`MonsterState` / `MonsterStateTransition`** — state machine components (with `DefaultStateID`).
*   **`RCG_MonsterAction` / `RCG_MonsterActionData`** — monster action definitions.
*   **`RCG_UnitSkillData`** — passive / leader skills.
*   **`RCG_UnitDataEditorPage`** — main editing page.

### A.5 Known Issues

*   `DeserializeFromJson` has a commented-out compatibility shim (`m_UnitSetting = m_MonsterData`) for legacy field migration.
*   "暫時用HP當作戰鬥力指標" / "Using HP as combat power for now" comment indicates the `m_CombatEffectiveness` fallback rule needs redesign.
