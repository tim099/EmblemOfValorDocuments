---
title: Max HP alter
description: Modify the target's max HP; permanent or this-battle-only scope
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Max HP alter

> Class: `RCG_MaxHPAlterSetting`

## Purpose
**Modify the target's max HP**. Examples:
*   Permanently raise a character's HP (battle reward / level-up)
*   Temporarily raise HP for this battle (post-battle revert)

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **AlterType** | Yes | Scope:<br>• **Permanently** — kept after battle<br>• **ThisBattle** — reverts at battle end |
| **Amount** | Yes | Delta (variable, can be negative). |
| **Target** | Yes | Target selector. |

## Behaviour
*   For non-dead targets → `target.AlterMaxHP(AlterType, Amount)`.
*   Description: "**{Permanently/ThisBattle} +{Amount} max HP to {Target}**" (i18n `MaxHPIcon_Des` with heart icon).
*   Awaits 0.6s for UI animation.

### Card fusion
Two cards fuse by adding `Amount` (**doesn't check AlterType equality** — potential gotcha).

## Notes
*   **Fusion AlterType mismatch**: Permanently + ThisBattle fused inherits the **first**'s AlterType — could mis-flag a permanent buff as temp.
*   **Negative Amount = lower HP cap**: legal; if current HP exceeds new cap behavior depends on `AlterMaxHP` impl (verify with team).
*   **Dead targets are skipped**: avoids buffing corpses.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MaxHPAlterSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_MaxHPAlterSetting` → "Max HP alter"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_AlterType` | AlterType | `AlterType` (file enum) | — | `Permanently` / `ThisBattle` |
| `m_Amount` | Amount | `IntVariable` | `MaxHP` | `[UCL_FieldOnGUI]` is commented out |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |

### A.3 Key methods
*   **`AddAction`** (async): per non-dead target → `aTarget.AlterMaxHP(m_AlterType, aAmount)`; awaits 0.6s.
*   **`Fusion`**: clone + `IntVariable.FuseAdd(m_Amount)`; **doesn't check `m_AlterType`**.
*   **`GetDescriptionFormat`** → i18n `MaxHPIcon_Des`, 4 params (Amount / Target / heart sprite / AlterType localize).

### A.4 Cross-system interactions
*   **`RCG_BattleUnit.AlterMaxHP(AlterType, int)`**: actual modification entry; handles HP / cap sync.
*   **`EffectIcon.Health`**: heart icon.

### A.5 Known issues
*   Legacy `CanEnhence / Enhence` commented out (enhancement system refactor).
