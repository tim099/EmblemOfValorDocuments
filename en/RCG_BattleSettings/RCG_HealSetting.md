---
title: Heal
description: Restores HP to selected targets; supports flat amount or percentage modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Heal

> Class: `RCG_HealSetting`

## Purpose
Restores HP to **selected targets**. The simplest "heal card", "healing skill", or "self-repair status" uses this.

## Inspector layout
```
▼ ✓ [Heal(Heal)] Restore 1 HP to (target)
    HealType    [Amount / Percentage]
    HealAmount  [Value] 1
    HealTarget  ▶ (target selector)
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **HealType** | Yes | Heal mode:<br>• **Amount** — restore a flat point count (e.g. "+5 HP").<br>• **Percentage** — restore a % of max HP (e.g. "+30% HP"). |
| **HealAmount** | Yes | Number (variable-bound). Treated as points for Amount, 0–100 for Percentage. |
| **HealTarget** | Yes | Target selector — self / all allies / chosen target / etc. |

> [!NOTE]
> **HealAmount** supports variables (e.g. "= remaining hand size", "= some status stacks"). Toggle the dropdown to switch between literal and variable.

## Behaviour

### What happens in battle
1. Resolve **HealTarget** to the actual targets.
2. For each target, restore HP per **HealType**.
3. UI shows green floating numbers.

### Description rendering
*   **HealType = Amount**: "Restore {amount} HP to {target}" (with heart icon).
*   **HealType = Percentage**: "Restore {amount}% HP to {target}".

### Card fusion
Two Heal cards fused together **add their HealAmount** (other fields take from the first). Example: "+3" + "+5" → "+8".

## Notes

*   **Don't combine Heal with attack on one card** — wrap them via **Combine effect** with separate children. Two orthogonal concerns shouldn't be jammed into one setting.
*   **Negative HealAmount**: technically legal but semantic nonsense. **For damage, use Attack** — don't abuse this.
*   **HealTarget = enemy**: legal but unusual (healing a foe is typically a debuff or joke card). Confirm intent.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_HealSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_HealSetting` → "Heal"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_HealType` | `HealType` | `HealType` (enum) | `HealType` | Values via `HealType_Amount` / `HealType_Percentage` keys |
| `m_HealAmount` | `HealAmount` | `IntVariable` | `HealAmount` | `[UCL_FieldOnGUI]` |
| `m_HealTarget` | `HealTarget` | `RCG_SelectTargetData` | `HealTarget` | Replaces deprecated `m_HealRange` |

### A.3 Key methods
*   **`AddAction`**: resolve targets → `RCG_PlayerHealAction(m_HealType, User, targets, m_HealAmount.GetValue, iData)` → `InsertAction`.
*   **`GetDescriptionFormat`**: `HealType.Amount` uses i18n `HealIcon_Des`; others call `m_HealType.GetLocalizeDes(...)`.
*   **`GetDescriptionShort`**: `Percentage` → `{amount}%`; else `{amount}`.
*   **`Fusion(other)`**: requires same type; clones and `IntVariable.FuseAdd` the amounts.

### A.4 Cross-system interactions
*   **`RCG_PlayerHealAction`**: actual Action class; handles VFX + HP modification.
*   **`RCG_SelectTargetData.GetTargets`**: shared target selector.
*   **`IntVariable`**: serializable numeric container with literal / variable / hidden-variable modes.

### A.5 Known issues
*   Legacy `m_HealRange` field and the related `DeserializeFromJson` migration are deprecated; current code is `m_HealTarget` only.
