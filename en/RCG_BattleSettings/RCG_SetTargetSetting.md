---
title: Set target
description: Auto-resolve and reassign targets via a rule (no UI); overrides existing target list
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Set target

> Class: `RCG_SetTargetSetting`

## Purpose
**Auto-pick targets via a rule** (no UI). Examples:
*   "Redirect attack to lowest-HP enemy"
*   "Redirect to back-row target"
*   AI auto-target chains

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **UseOldSelectUnitRule** | — | Default checked (legacy compat). Checked = use legacy `RCG_SelectTargetData`; unchecked = new `RCG_TargetSelectRule`. |
| **SelectTarget** | When UseOldSelectUnitRule=true | Legacy rule (`RCG_SelectTargetData`). |
| **SelectUnitRule** | When UseOldSelectUnitRule=false | New rule (`RCG_TargetSelectRule`). |

## Behaviour
*   Resolves the rule auto-magically, overwrites `iData.Targets`.
*   No UI / no input — fits AI and automation.
*   Description: short name from the active rule.

## Notes
*   **vs Select target**: this setting **doesn't show UI**; **Select target** prompts the player.
*   **New rule vs legacy rule**: the new `RCG_TargetSelectRule` is more expressive; the legacy `RCG_SelectTargetData` is kept for compat. Prefer new (uncheck `UseOldSelectUnitRule`).
*   **Source TODO**: `// TODO: Test QWQ2` — new rule needs verification.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SetTargetSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_SetTargetSetting` → "Set target"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_UseOldSelectUnitRule` | UseOldSelectUnitRule | `bool` | — | Default `true` (compat) |
| `m_SelectUnitRule` | SelectUnitRule | `RCG_TargetSelectRule` | — | `[Conditional("m_UseOldSelectUnitRule", false)]` |
| `m_SelectTarget` | SelectTarget | `RCG_SelectTargetData` | — | `[Conditional("m_UseOldSelectUnitRule", true)]` |

### A.3 Key methods
*   **`AddAction`** (async): per `m_UseOldSelectUnitRule`, calls `m_SelectUnitRule.SelectTargets(iData)` or `m_SelectTarget.GetTargets(iData)`, assigns to `iData.Targets`.
*   **`GetDescription`**: delegates to the active rule's `Description / GetShortName`.

### A.4 Cross-system interactions
*   **`RCG_TargetSelectRule`**: new rule container.
*   **`RCG_SelectTargetData`**: legacy selector, widely used by other settings.

### A.5 Known issues
*   `// TODO: Test QWQ2` — new rule pending validation.
