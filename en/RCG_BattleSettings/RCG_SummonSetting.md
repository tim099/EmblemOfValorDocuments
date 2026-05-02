---
title: Summon
description: Summon units to the battlefield (enemy or ally); supports default and dead-position modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Summon

> Class: `RCG_SummonSetting`

## Purpose
**Summon units to the battlefield**. Examples:
*   Summon ally support (healers, buffers, tanks)
*   Enemy summons minions
*   "Resurrect at dead unit's position" mechanics

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **UnitData** | Yes | Summon data; embedded `RCG_UnitSummonData` with summon type, unit, monster flag. |

> [!NOTE]
> The nested `RCG_UnitSummonData` holds the real config:
> *   `m_Unit` — unit type to summon
> *   `m_IsMonster` — default faction (**auto-flipped when triggered by enemy**)
> *   `m_SummonType` — `Default` (find empty slot) / `UnitDeadPosiiton` (resurrect on a dead unit's slot)
> *   `m_Pos` — summon position (front / back / All)

## Behaviour
*   **Faction flip**: if triggered by enemy faction, `IsMonster` is flipped (so monster summoning ally = player-perspective enemy).
*   **Default mode**: prefer `iData.TargetPositions[0]` empty slot; auto-locate otherwise.
*   **UnitDeadPosiiton mode**: summon at a dead unit's position (revival / corpse-spawn mechanic).
*   Plays `VFX_Summon` + summon anim → triggers the new unit's `OnBattleStart` (entrance effects).
*   Description: `SummonDes_{Monster|Player|UnitDeadPosiiton}` + unit name.

## Notes
*   **Silent failure on no slot**: `Debug.LogError` only — card may seem ineffective.
*   **Summon cap**: limited by max units per side.
*   **OnBattleStart triggers**: any "on enter" abilities fire.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SummonSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_SummonSetting` → "Summon"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_UnitData` | UnitData | `RCG_UnitSummonData` | `UnitData` | `[AlwaysExpendOnGUI]` |

### A.3 Key methods
*   **`AddAction`** (async):
    1. Flip `aIsMonster` if `iData.Faction == UnitFaction.Enemy`.
    2. Determine spawn position per `m_SummonType` (Default → empty slot / UnitDeadPosiiton → dead slot).
    3. `RCG_BattleField.SpawnUnit` + `VFX_Summon` + `SummonAnim` await + `OnBattleStart`.
*   **`LocalizeKey` (private)**: per SummonType + IsMonster picks i18n key.
*   **`Info / GetDescriptionFormat`**: shows summoned unit info + description.

### A.4 Cross-system interactions
*   **`RCG_UnitSummonData`**: summon data (unit / type / position / faction).
*   **`RCG_BattleField.SpawnUnit / TryGetEmptyPositions`**: spawn entry.
*   **`CommonVFX.VFX_Summon`**: summon VFX.
*   **`RCG_Unit.SummonAnim / OnBattleStart`**: entrance sequence.

### A.5 Known issues
*   Legacy `TriggerAsync` and partial summon logic kept as comments.
