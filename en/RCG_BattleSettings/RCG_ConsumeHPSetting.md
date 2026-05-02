---
title: Consume HP
description: Reduce target HP (flat / current-HP %, max-HP %); used for self-damage effects
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Consume HP

> Class: `RCG_ConsumeHPSetting`

## Purpose
**Reduce target HP** (not treated as an attack, so **armor doesn't apply**). Examples:
*   "Self-damage 5 HP for massive resource gain"
*   "Consume 30% current HP to trigger a powerful effect"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **ConsumeType** | Yes | Mode:<br>• **CurHPPercentage** — % of current HP (default)<br>• **MaxHPPercentage** — % of max HP<br>• **Value** — flat amount |
| **Amount** | Yes | Consumed value (variable). 0–100 for percentage modes. |
| **Target** | Yes | Target selector (typically self). |

## Behaviour
*   Computes consumed amount per ConsumeType:
    *   `CurHPPercentage` → `RoundToInt(0.01 * Amount * currentHP)`
    *   `MaxHPPercentage` → `RoundToInt(0.01 * Amount * maxHP)`
    *   `Value` → `Amount` directly
*   Calls `target.UnitHeal(-aConsumeAmount)`.
*   Logs to `RCG_BattleManager.Ins.m_BattleStats.LogStatsCostHealth(...)`.
*   Description: `ConsumeHP_{ConsumeType}_Des` (with heart icon).

## Notes
*   **Percentage rounding**: `RoundToInt`, so edge values (e.g. 1% × 50 HP = 0.5) round to 0 or 1. Design for **integer-predictable** outcomes.
*   **Won't insta-kill outright**: if HP reaches 0 it'll trigger normal death; for forced kills use **Instant death**.
*   **Not an attack**: armor / counter / damage-mod tags don't apply — pure HP loss.
*   **Targeting enemies**: legal but odd (this is typically self-targeted); for damage prefer **Attack**.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConsumeHPSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ConsumeHPSetting` → "Consume HP"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_ConsumeType` | ConsumeType | enum | — | 3 modes |
| `m_Amount` | Amount | `IntVariable` | `Amount` | `[UCL_FieldOnGUI]`; default 50 |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |

### A.3 Key methods
*   **`AddAction`**: branches on ConsumeType, computes `aConsumeAmount`, calls `aTarget.UnitHeal(-aConsumeAmount)`, then `LogStatsCostHealth`.
*   **`GetDescriptionFormat`**: i18n `ConsumeHP_{ConsumeType}_Des` with heart sprite.
*   **`GetDescriptionShort`** → `m_Amount.GetDes(true)`.

### A.4 Cross-system interactions
*   **`RCG_BattleUnit.UnitHeal(int)`**: shared heal entry; negative values = damage.
*   **`RCG_BattleManager.m_BattleStats.LogStatsCostHealth`**: stats logging.
