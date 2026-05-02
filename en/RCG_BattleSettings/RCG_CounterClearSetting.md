---
title: Counter clear
description: Clear all counters on the trigger source (equipment / unit skill) at once
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Counter clear

> Class: `RCG_CounterClearSetting`

## Purpose
**Clear ALL counters at once** on the trigger source — stronger than Counter alter: zeros every counter rather than one. Examples:
*   "Reset all accumulated stacks after triggering equipment ability"
*   "Force-clear equipment charge under specific conditions"

> [!IMPORTANT]
> Only valid **inside equipment / unit-skill trigger effects** (same constraint as Counter alter).

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CounterEffectType** | Yes | `UnitSkill` or `Equipment` — **for description display only**; the actual clear targets all counters on the source regardless of type. |

## Behaviour
*   Calls `counter.ClearAllCounters()` — clears **every** counter on the trigger source.
*   Description: "**Clear {CounterEffectType} counters**" (i18n `CounterClearDes`).

## Notes
*   **No CounterId field**: clear is all-or-nothing. To zero one specific counter use **Counter alter** + `Set 0`.
*   **EffectTriggerSource mismatch logs error**: same as Counter alter.
*   **Always expanded**: `[AlwaysExpendOnGUI]`.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterClearSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_CounterClearSetting` → "Counter clear"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CounterEffectType` | CounterEffectType | enum | — | Shared with `RCG_CounterAlterSetting` |

### A.3 Key methods
*   **`AddAction`** (async): casts `iData.EffectTriggerSource as RCGI_EffectCounter` and calls `ClearAllCounters()`.
*   **`GetDescriptionFormat / GetDescriptionShort`**: i18n `CounterClearDes` with `m_CounterEffectType.GetLocalizeName()`.

### A.4 Cross-system interactions
*   **`RCGI_EffectCounter.ClearAllCounters`**: clearing entry.
