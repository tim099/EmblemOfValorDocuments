---
title: Battle tags
description: Attaches battle tags (Brittle, Agile, ...) for tag aggregation; produces no actions itself
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle tags

> Class: `RCG_BattleTagSetting`

## Purpose
**Attaches battle tags** to an effect — produces **no battle actions**, just acts as a "**tag carrier**". Used for:
*   Tagging a Combine effect with **Brittle**, **Agile**, **Consume**, etc.
*   Sourcing terminology entries on the card info tooltip

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **BattleTags** | Yes | List of `RCG_BattleTagGenData` (tag templates). Multiple allowed. |

## Behaviour
*   **No Action**: `AddAction` is empty — this setting only declares tags.
*   **Card info**: each tag shows its name + description on the tooltip; stackable tags (`m_IsStackable=true`) include the stack count in their description.
*   **Short description**: concatenates all tags' short names without extras.
*   **`GetBattleTags()`**: container types (Combine, Loop, Conditional, ...) call this to aggregate tags upward.

## Notes
*   **Want actual gameplay effect? Use Combine or Status**: this setting is metadata only. "Brittle" damage reduction needs separate trigger logic.
*   **Empty list**: legal but does nothing — wasted data.
*   **Always expanded**: `[AlwaysExpendOnGUI]` keeps it open in the Inspector.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleTagSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_BattleTagSetting` → "Battle tags"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_BattleTags` | BattleTags | `List<RCG_BattleTagGenData>` | — | |

### A.3 Key methods
*   **`AddAction`** → empty (commented `// base.AddAction(...)`).
*   **`GetBattleTags()`** → `m_BattleTags.Clone()`; aggregated by container types.
*   **`Infos`**: per tag → `{LocalizedName}\n{Description}`; stackable types get `StackedBattleTag` template with `eBattleVariable_BattleTagStackCount`.
*   **`GetShortName`** → concat per-tag short names; falls back to base when empty.
*   **`GetDescription`** → always `string.Empty` (intentional override).

### A.4 Cross-system interactions
*   **`RCG_BattleTagGenData / RCG_BattleTag`**: tag template + instance.
*   **i18n keys**: `StackedBattleTag`, `eBattleVariable_BattleTagStackCount`, tag color `RCG_BattleTag`.
*   **Callers of `GetBattleTags`**: all container types (Combine, Loop, Conditional, Foreach, ...).
