---
title: Monster Action Data (RCG_MonsterActionData)
description: Template for a single monster action / move — target selection, effects, icon, description
last_invoked: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Monster Action Data

> Class name: `RCG_MonsterActionData`

## Purpose

**Template for a single monster action / move**. Examples: "Slash", "Summon Followers", "Self-destruct". Each Action contains:
*   Target selection rule (which units can be selected as targets)
*   Effects (damage, buff, summon, etc.)
*   Icon
*   Description (auto / manual)

`RCG_UnitData.MonsterStates` references a set of `RCG_MonsterActionData` as the action pool usable in that state.

Inherits from `RCG_Asset<RCG_MonsterActionData>`.

## Editor Layout

```
RCG_MonsterActionData: <ID>
    Action (m_Action)
        SelectUnitRule         ← target selection rule
        Effect structure       ← actual effects
        Icon                   ← preview icon
        OverrideDescription    ← manually override the auto description
        UnitAction             ← actual action to execute
        Infos                  ← additional info shown in tooltip
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Action** | yes | Inner `RCG_MonsterAction`, contains all data for this action |

`RCG_MonsterAction` contains:

| Sub-field | Description |
|---|---|
| **SelectUnitRule** | Target selection rule (`RandomEnemy` / `Friend` / self / position N, etc.) |
| **OverrideDescription** | Whether to use `m_UnitAction`'s description on top of auto |
| **UnitAction** | The actual action (damage formula, effect sequence) |
| **Icon** | Preview / tooltip icon |
| **SkillLevel** | Action level (set at runtime by `RCG_MonsterLevelActionData.GetAction`) |

## Behavior

### Target Selection → Trigger → Description
1. After the monster picks this Action in battle, find targets via `SelectUnitRule`.
2. Apply `m_UnitAction` effects to targets.
3. UI tooltip shows `Icon` + `GetDescription()` + `Infos` for additional tag info.

### Auto vs Override Description
When `OverrideDescription = true`, preview shows the main description **plus** the appended `m_UnitAction.GetDescription()` (both segments displayed).

### Default ID
`RCG_MonsterActionGenData.IdleID` (`"Idle"`) is the "no action" default; any unit without an action defined will use this.

## Caveats

*   **SkillLevel set externally**: `RCG_MonsterLevelActionData.GetAction(level)` writes the level to `m_SkillLevel`; this data is just a template — runtime uses a clone.
*   **OverrideDescription dual display**: may make tooltips overly long; disable when not needed.
*   **Don't delete the Idle action**: many fallback paths return `RCG_MonsterActionGenData.Idle.GetData()`; missing this ID causes NREs.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MonsterActionData.cs`
*   **Inherits**: `RCG_Asset<RCG_MonsterActionData>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Action` | Action | `RCG_MonsterAction` | Main data container |

### A.3 Key Methods

*   **`Preview`** — editor rendering: icon + selection rule + description + Effects description (if OverrideDescription) + Infos.
*   Constructor defaults to `ID = "New Action"`.

### A.4 System Interactions

*   **`RCG_MonsterAction`** — main data model; contains `m_SelectUnitRule` / `m_UnitAction` / `m_Icon` / `m_SkillLevel`.
*   **`RCG_MonsterActionGenData`** (in-file) — Asset Entry; `Idle` is the system default.
*   **`RCG_MonsterLevelActionData`** — wraps Actions with level-tier support.
*   **`RCG_UnitData.m_MonsterStates`** — container that references Actions.

### A.5 Known Issues

*   Commented-out `DeserializeFromJson` migration shim (`m_ID == "AttackEffect"` → `DefaultID`) shows legacy ID migration history.
