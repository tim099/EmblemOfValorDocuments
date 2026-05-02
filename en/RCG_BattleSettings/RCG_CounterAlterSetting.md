---
title: Counter alter
description: Modify the trigger source's (equipment / unit skill) internal counter values
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Counter alter

> Class: `RCG_CounterAlterSetting`

## Purpose
**Modify the trigger source's counters** — internal numeric trackers on equipment or unit skills (e.g. "uses remaining", "accumulated stacks"). Examples:
*   "Equipment: triggers every 3 hits taken" (counter cycles)
*   "Equipment: 3 uses per battle" (counter as charge tracker)

> [!IMPORTANT]
> Only valid **inside equipment / unit-skill trigger effects** — relies on `iData.EffectTriggerSource` implementing `RCGI_EffectCounter`. Generic card triggers have no counter to modify.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CounterEffectType** | Yes | Counter scope: `UnitSkill` or `Equipment`. |
| **CounterId** | Yes | Counter index (0, 1, 2, ... for multi-counter equipment). |
| **CounterAlter** | Yes | Delta (variable). |
| **CounterAlterType** | Yes | Mode: `Add` / `Sub` / `Set`. |

## Behaviour
*   Reads `iData.EffectTriggerSource`; if it implements `RCGI_EffectCounter` → `counter.AlterCounter(CounterId, CounterAlterType, CounterAlter)`.
*   Description: "**{CounterEffectType} counter {Id+1} {CounterAlterType} {CounterAlter}**" (i18n `CounterAlterDes`).

## Notes
*   **EffectTriggerSource mismatch logs an error**: in card contexts, console gets an error but no crash. **Use only inside equipment / skill effects**.
*   **CounterId out of range**: triggers undefined behavior — verify ID matches the equipment's counter count.
*   **Always expanded**: `[AlwaysExpendOnGUI]`.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CounterAlterSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_CounterAlterSetting` → "Counter alter"
*   **Same-file enums**: `CounterEffectType { UnitSkill, Equipment }`, `CounterAlterType { Add, Sub, Set }`

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CounterEffectType` | CounterEffectType | enum | — | |
| `m_CounterAlterType` | CounterAlterType | enum | — | |
| `m_CounterId` | CounterId | `int` | — | Default 0 |
| `m_CounterAlter` | CounterAlter | `IntVariable` | — | |

### A.3 Key methods
*   **`AddAction`** (async): casts `iData.EffectTriggerSource as RCGI_EffectCounter` and calls `AlterCounter(m_CounterId, m_CounterAlterType, m_CounterAlter.GetValue(iData))`.
*   **`GetDescriptionShort`** → returns i18n `RCG_CounterAlterSetting` literal label.
*   Legacy `m_TriggerData.m_Equipment / m_UnitSkill` branch logic is commented out.

### A.4 Cross-system interactions
*   **`RCGI_EffectCounter`**: counter interface implemented by equipment / unit skill.
*   **`iData.EffectTriggerSource`**: trigger source instance.
