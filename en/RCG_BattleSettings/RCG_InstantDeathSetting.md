---
title: Instant death
description: Kill targets instantly; supports unconditional or "if HP below threshold" modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Instant death

> Class: `RCG_InstantDeathSetting`

## Purpose
**Kills targets instantly** (ignores HP / armor). Examples:
*   "Execute" / "Decapitate" cards: kill targets below X HP
*   Special story "unconditional kill" effects (cutscene / curse)

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Type** | Yes | Mode:<br>• **Default** — unconditional kill on all targets<br>• **HP** — kill only if target HP ≤ specified value |
| **Target** | Yes | Target selector. |
| **HP** | When Type=HP | HP threshold; targets with HP ≤ this die. Conditional on type. |

## Behaviour
*   **Default**: calls `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)` (with term-style trigger).
*   **HP**: per target evaluates `target.HP <= m_HP` independently → `target.InstantDeath()` if matching.
*   Description:
    *   `Default` → "**Instant death on {target}**"
    *   `HP` → "**Below {HP} HP, instant death on {target}**"

### Card info
Shows the `Term.InstantDeath` term tooltip (explains the mechanic).

## Notes
*   **Default mode is very strong**: also kills bosses (unless they have an "ignore insta-death" status). Use sparingly.
*   **HP mode is fair on AOE**: per-target evaluation prevents "AOE that only kills the first target".
*   **Negative HP**: `m_HP < 0` never matches (target HP can't go below 0).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_InstantDeathSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_InstantDeathSetting` → "Instant death"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Type` | Type | `InstantDeathType` (file enum) | — | `Default` / `HP` |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |
| `m_HP` | HP | `IntVariable` | `HP` | `[Conditional(nameof(m_Type), false, HP)]`; default 1 |

### A.3 Key methods
*   **`AddAction`**: branches on `m_Type`:
    *   `Default` → `RCG_TermInstantDeath.TriggerInstantDeath(iData, targets)`.
    *   `HP` → per target check `target.HP <= m_HP.GetValue(iData)` → `target.InstantDeath()`.
*   **`Info`** → `new CardInfoData(RCG_Term.GetTerm(Term.InstantDeath))`.
*   **`GetDescriptionFormat`**: `InstantDeath_Des` (Default) / `InstantDeath_HP_Des` (HP).

### A.4 Cross-system interactions
*   **`RCG_TermInstantDeath.TriggerInstantDeath`**: term-trigger entry with VFX.
*   **`RCG_BattleUnit.InstantDeath`**: direct HP=0 + death flow.
*   **`Term.InstantDeath`**: matching term enum.

### A.5 Known issues
*   `Condition` mode (`m_Conditions`) is commented out — for "conditional insta-death" use Conditional + InstantDeath(Default).
