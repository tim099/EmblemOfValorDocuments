---
title: Defense
description: Modify armor: add / sub / clear / double / decay / set
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Defense

> Class: `RCG_DefenseSetting`

## Purpose
**Modify armor stacks**. Core "defense card" / "buff card" mechanic with multiple operation modes.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **DefenseAlterType** | Yes | Mode:<br>• **Add** — gain armor<br>• **Sub** — lose armor<br>• **Clear** — force-clear<br>• **Double** — double current<br>• **Decay** — natural decay (**can be blocked by certain statuses**, unlike Clear)<br>• **Set** — set to specific value |
| **Defense** | When Add/Sub/Set | Armor amount (variable). Auto-hidden in Clear/Double/Decay modes. |
| **DefenseTarget** | Yes | Target selector (replaces deprecated `m_Range`). |

## Behaviour
*   Apply per DefenseAlterType:
    *   `Add` / `Sub` / `Set` — direct numeric change
    *   `Clear` — force zero (**not blockable**)
    *   `Decay` — natural decay (**blockable by statuses like "shield retain"**)
    *   `Double` — multiply current armor
*   Description per mode: i18n keys `DefIcon_DesAdd` / `DefIcon_DesSub` / `DefIcon_DesClear` / `DecayDes` / etc., with armor icon.

## Notes
*   **Decay vs Clear**: Clear is forced-zero; Decay can be blocked by anti-decay statuses. Use `Decay` for end-of-turn natural decay.
*   **Sub clamps at 0**: armor floor is 0; over-subtracting just stops at 0, not negative.
*   **Always expanded**: `[AlwaysExpendOnGUI]`.
*   **vs Status**: armor is its own value, not a status stack. Stack-based conditions don't see armor.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DefenseSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_DefenseSetting` → "Defense"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_DefenseAlterType` | DefenseAlterType | enum (file) | — | 6 modes |
| `m_Defense` | Defense | `IntVariable` | `Defense` | `[UCL_FieldOnGUI] + [Conditional(... Add, Sub, Set)]` |
| `m_DefenseTarget` | DefenseTarget | `RCG_SelectTargetData` | — | Replaces deprecated `m_Range` |

### A.3 Key methods
*   **`AddAction`** (beyond shown range): per `DefenseAlterType`, applies on `m_DefenseTarget.GetTargets(iData)`.
*   **`GetDescriptionFormat`**: per mode → `DefIcon_DesXxx` / `DecayDes` etc.
*   **`Infos`**: when `IsShowOnUI`, adds armor icon entry.

### A.4 Cross-system interactions
*   **`EffectIcon.Armor`**: armor icon sprite.
*   **`RCG_BattleUnit` armor pool**: actual storage; decay can be blocked by `m_UnitStatus` buffs.
