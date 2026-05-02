---
title: Battle tag combine
description: Combine effect specialization that wraps a prefix BattleTag as a trigger condition
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle tag combine

> Class: `RCG_BattleTagCombineSetting`

## Purpose
A specialization of **Combine effect**. Adds a **prefix BattleTag** that acts as a **condition + trigger timing**: the main content runs only when the specified `TriggerOn` fires with these tags present.

Examples:
*   "If this card has [Consume] tag, on play trigger X effect"
*   "If has [Agile], on draw trigger Y effect"

Inherits from `RCG_CombineSetting` (so all Combine fields apply too).

## Key fields (in addition to Combine's)

| Inspector label | Required | Notes |
|---|---|---|
| **TriggerOn** | Yes | Trigger timing: `OnPlay` (default) / `OnDraw` / etc. (see `TriggerOn` enum). |
| **PrefixBattleTagSetting** | Yes | Prefix BattleTag setting — main content fires only when these tags trigger. |

Plus inherited: **OverrideDescription** / **Description** / **CombineSettings**.

## Behaviour
*   `CombineSettings` puts `PrefixBattleTagSetting` first as the "condition", followed by the original list.
*   Description: "**If {prefix tags}, then {main effect}**" (i18n `IfCardConditions2` + `ConditionFitThenDes`).
*   On trigger: first calls `OnTriggerEffect(TriggerOn)` on each prefix tag, then runs the rest.
*   `GetBattleTags()` **excludes** the prefix tags — they're a condition, not main-effect tags.

## Notes
*   **TriggerOn must match the card's actual fire point**: if the card fires `OnPlay` but this is set to `OnDraw`, the main effect never runs.
*   **Empty PrefixBattleTagSetting**: results in "If , then ..." — data error.
*   **vs Conditional**: Conditional uses `RCG_Condition`s; this setting uses **TriggerOn + tags** as the precondition. More precise but narrower.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagCombineSetting.cs`
*   **Inherits**: `RCG_CombineSetting`
*   **No i18n class key**: editor shows stripped name `BattleTagCombine`

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_TriggerOn` | TriggerOn | `TriggerOn` (enum) | — | Default `OnPlay` |
| `m_PrefixBattleTagSetting` | PrefixBattleTagSetting | `RCG_BattleTagSetting` | — | Nested |
| (inherited) | (Combine fields) | — | — | See parent |

### A.3 Key methods
*   **`CombineSettings` (override)**: prepends `m_PrefixBattleTagSetting` then base.CombineSettings.
*   **`GetBattleTags`** (override): **skips** prefix; only aggregates `m_CombineSettings.GetEnableBattleSettings()` tags.
*   **`AddAction`**: for `i==0` (prefix) → `OnTriggerEffect(m_TriggerOn, iData)` per tag; others → normal `AddAction(InsertInOrder)`.
*   **`GetDescriptionFormat / GetDescription / GetDescriptionShort`**: wrap with `IfCardConditions2` + `ConditionFitThenDes`.

### A.4 Cross-system interactions
*   **`RCG_BattleTagSetting`**: wrapped as prefix.
*   **`RCG_EffectTriggerOn.GetEffectTriggerOn(TriggerOn)`**: enum → triggerable object.
*   **i18n keys**: `IfCardConditions2`, `ConditionFitThenDes`.

### A.5 Known issues
*   No `AllTypes` registration entry for class i18n; dropdown shows stripped name.
