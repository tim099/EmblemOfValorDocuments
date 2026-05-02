---
title: Random trigger
description: Three random modes — random pick of N effects, batch by chance, or batch by variable chance
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Random trigger

> Class: `RCG_RandomTriggerSetting`

## Purpose
**Adds randomness to effects**. Three modes:
*   **RandomPick** — randomly select N effects from a list to fire
*   **TriggerByRate** — flat percentage decides whether the entire batch fires
*   **TriggerByVariableRate** — same but the percentage comes from a variable (e.g. "based on current HP ratio")

Used for gambling-style cards, probabilistic buffs, multiple-choice randomness.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **RandomTriggerMode** | Yes | Three modes (see below). |
| **RandomPickCount** | RandomPick mode | How many effects to pick. |
| **TriggerRate** | TriggerByRate mode | Trigger chance (0–100, slider). |
| **VariableRate** | TriggerByVariableRate mode | Variable holding the percentage (0–100 range). |
| **TriggerSettings** | Yes | Candidate effect list; only `IsEnable = true` entries are used. |

### Mode details
*   **RandomPick** — randomly picks N from enabled candidates; same effect doesn't fire twice.
*   **TriggerByRate** — rolls 0–99 once, fires the **whole batch** if `< TriggerRate`; otherwise none.
*   **TriggerByVariableRate** — same as TriggerByRate but rate comes from `VariableRate.GetValue`.

## Behaviour
*   Description lists all candidates with the mode info (e.g. "50% chance: [A], [B]" or "Random pick 1: [A] [B] [C]").

## Notes
*   **RandomPickCount > candidates**: `RandomPick(list, count)` typically caps at the candidate count.
*   **Empty list**: legal but does nothing.
*   **TriggerByRate at 0 / 100**: never / always — usually a design error.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_RandomTriggerSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_RandomTriggerSetting` → "Random trigger"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_RandomTriggerMode` | RandomTriggerMode | enum | — | 3 modes |
| `m_RandomPickCount` | RandomPickCount | `IntVariable` | — | `[Conditional(nameof(m_RandomTriggerMode), false, RandomPick)]` |
| `m_TriggerRate` | TriggerRate | `int` | — | `[UCL_IntSlider(0, 100)] + [Conditional(... TriggerByRate)]`; default 50 |
| `m_VariableRate` | VariableRate | `IntVariable` | — | `[Conditional(... TriggerByVariableRate)]` |
| `m_TriggerSettings` | TriggerSettings | `List<RCG_BattleSetting>` | — | Filtered: `m_TriggerSettings.Where(IsEnable)` |

### A.3 Key methods
*   **`AddAction`**: branches on `m_RandomTriggerMode`:
    *   `RandomPick` → `RCG_GameManager.Random.RandomPick(allEnabled, count)`, AddAction each in order.
    *   `TriggerByRate` → `Random.Range(0, 100) < m_TriggerRate` gate; if true, fire all.
    *   `TriggerByVariableRate` → same with rate from `m_VariableRate.GetValue(iData)`.
*   **`GetFusionCandidateSettings`** → aggregates enabled candidates' candidates (not self).
*   **`GetFusionBaseSetting`**: clones + rebuilds `m_TriggerSettings` as placeholders.

### A.4 Cross-system interactions
*   **`RCG_GameManager.Random.RandomPick / Range`**: seeded random source (affects reproducibility).
