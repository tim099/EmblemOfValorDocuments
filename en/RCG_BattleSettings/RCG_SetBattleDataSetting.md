---
title: Set battle data
description: Modify battle-scope shared variables (battle-internal data store)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Set battle data

> Class: `RCG_SetBattleDataSetting`

## Purpose
**Modify battle-scope shared data** (`RCG_BattleManager.BattleData`) — a numeric container shared across settings within this battle. Like "battle-internal global variables". Examples:
*   "Track total damage dealt"
*   "Record how many times an action fired this battle"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **RuntimeDataOperator** | Yes | Embedded `RCG_RuntimeDataSetter` (designates field + value + computation). Default bound to `RCG_RuntimeStructGenData.BattleDataID` (battle data struct). |

## Behaviour
*   On trigger → `RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)` writes the computed result to the named field.
*   Description fully delegated to `m_RuntimeDataOperator.GetDescription`.
*   `GetShortName` defaults to `"SetBattleData:{operator.GetShortName()}"`.

## Notes
*   **Data struct must be defined upfront**: which fields `RuntimeDataOperator` can target depends on the `RCG_RuntimeStructGenData.BattleDataID` struct definition.
*   **Doesn't fuse**: this setting is excluded from card fusion (`GetFusionBaseSetting` returns null).
*   **Battle-end resets**: BattleData is per-battle; for cross-battle persistence use **Resource alter** or global RuntimeData.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetBattleDataSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_SetBattleDataSetting` → "Set battle data"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_RuntimeDataOperator` | RuntimeDataOperator | `RCG_RuntimeDataSetter` | — | Default `RCG_RuntimeStructGenData.BattleDataID` |

### A.3 Key methods
*   **`AddAction`**: `RCG_BattleManager.BattleData.SetValue(m_RuntimeDataOperator)`.
*   **`GetDescription / GetDescriptionFormat / GetShortName`**: delegate to `m_RuntimeDataOperator`.
*   **`GetFusionCandidateSettings`** → empty; **`GetFusionBaseSetting`** → null.

### A.4 Cross-system interactions
*   **`RCG_BattleManager.BattleData`**: battle-scope shared data store.
*   **`RCG_RuntimeDataSetter`**: field setter with formula.
*   **`RCG_RuntimeStructGenData.BattleDataID`**: data struct template.
