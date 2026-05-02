---
title: Battle State Data (RCG_BattleStateData)
description: Battle "phase" (state) definition — what runs on entering this state, what comes next
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle State Data

> Class name: `RCG_BattleStateData`

## Purpose

**Definition of a battle "phase (state)"** — e.g., "Player Turn", "Enemy Turn", "Damage Settlement", "Turn End". Each state runs a fixed sequence on entry (draw cards, settle buffs, fire turn events, etc.). The whole battle flows between these states.

Inherits from `RCG_Asset<RCG_BattleStateData>`.

## Editor Layout

```
RCG_BattleStateData: <ID>
    Name                ← display name (localized)
    NextState           ← ID of the next state
    EnterStateActions   ← sequence of BattleSettings to run on entering this state
    StateData           ← state's own runtime data
    BattleState         ← corresponding BattleManager.BattleState enum value
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | no | Display name (localized); falls back to ID when empty |
| **NextState** | yes | The state to transition to after this one — the state machine skeleton |
| **EnterStateActions** | no | Sequence of `RCG_BattleSetting` to fire on entering this state |
| **StateData** | — | The state's internal `RCG_RuntimeData` (variable storage) |
| **BattleState** | — | Corresponding `RCG_BattleManager.BattleState` enum value (None / PlayerTurn / EnemyTurn, etc.) |

## Behavior

### State Machine Flow
1. On entering this state → `OnEnterState(data)` runs all `EnterStateActions` in order (filters out `Enable=false`).
2. While in the state → externally driven via `StateUpdate()` (currently empty).
3. On exit → `ExitState()` (currently empty).
4. Then transitions to `m_NextState`.

### State and BattleState Mapping
The `State` (property) converts `m_BattleState.DefaultValue` (string) into `RCG_BattleManager.BattleState` enum; falls back to `None` on missing. **This mapping lets data-driven states correspond to programmatic enum logic.**

## Caveats

*   **`StateUpdate` / `ExitState` are empty**: currently driven externally; these hooks reserved for future extension.
*   **`m_StateActions` deprecated**: replaced by `m_EnterStateActions`; legacy data **does not auto-migrate** on deserialize (the `m_EnterStateActions = m_StateActions.Clone();` line is commented out).
*   **`m_State` enum field deprecated**: replaced by `m_BattleState` (`RCG_RuntimeDataConst`); the legacy field still exists in the base class but is no longer written.
*   **`EnterStateActions` filter**: filters out `Enable=false` settings; can be toggled at edit time.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleStateData.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleStateData>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_NextState` | NextState | `RCG_BattleStateGenData` | |
| `m_EnterStateActions` | EnterStateActions | `List<RCG_BattleSetting>` | |
| `m_StateData` | StateData | `RCG_RuntimeData` | |
| `m_BattleState` | BattleState | `RCG_RuntimeDataConst` | Default `BattleState` type |

### A.3 Key Methods

*   **`OnEnterState(TriggerEffectData, AddActionMode)`** — enters state; runs all `EnterStateActions`.
*   **`StateUpdate()`** — empty stub.
*   **`ExitState()`** — empty stub.
*   **`State` (property)** — fetches enum value from `m_BattleState.DefaultValue`.
*   **`LocalizeName`** — `m_Name.HasLocalize ? m_Name.Name : ID`.
*   **`EnterStateActions` (property)** — filters `Enable=true` actions.

### A.4 System Interactions

*   **`RCG_BattleManager.BattleState` (enum)** — corresponding internal state.
*   **`RCG_BattleStateGenData`** — Asset Entry wrapper.
*   **`RCG_BattleSetting`** — element of `EnterStateActions`.
*   **`RCG_RuntimeData / RCG_RuntimeDataConst`** — data containers.

### A.5 Known Issues

*   `m_StateActions` and `m_State` enum deprecated; no migration path on deserialize.
