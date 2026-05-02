---
title: Diminish (disable card effects)
description: Weakens hand cards by removing N leaf effects, prioritizing enhancements first
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Diminish (disable card effects)

> Class: `RCG_DiminishSetting`

## Purpose
**Weakens selected hand cards** — removes N "leaf effects" (single atomic actions like Attack / Heal / Draw). **Enhancements are consumed first**, then base effects. Examples:
*   Enemy skill: "Halve the effect of your next card"
*   Curse cards: "Negate one effect on the target card"

Removed leaves are replaced by `RCG_DiminishedPlaceholder` (red "Diminished" text).

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectHandCardSetting** | Yes | Card selector. |
| **DiminishEffectCount** | Yes | Leaves to remove (default 1). |

## Behaviour
*   After selection, per card → `aCard.Diminish(token, DiminishEffectCount)`.
*   Removal order: **enhancements first**, then base effects.
*   Removed leaves become `RCG_DiminishedPlaceholder` (red text "Diminished").

## Notes
*   **Diminish is one-way**: replaced placeholders **cannot be diminished further** (avoids infinite stacking).
*   **Leaf effects aren't always obvious**: Combine / Conditional contain multiple leaves; `DiminishEffectCount = 1` may not target the "key one".
*   **Large counts**: exceeding total leaves replaces the whole card with placeholders. Tune carefully.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_DiminishSetting` → "Diminish"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_DiminishEffectCount` | DiminishEffectCount | `int` | — | `[SerializeField] protected`; default 1 |

### A.3 Key methods
*   **`AddAction`** (async): select → per card `aCard.Diminish(iToken, m_DiminishEffectCount)`.
*   **`GetBattleTags`** → empty list (diminished cards don't aggregate tags).
*   **`GetDescription`**: "{selector desc}\n{diminish info} ({count})" via i18n `DiminishSettingDes`.
*   **`GetDescriptionParams`**: includes `Title`, `DC` (count string), and selector child params.

### A.4 Cross-system interactions
*   **`RCG_CardBattleData.Diminish(token, count)`**: actual removal entry; consumes enhancement leaves first.
*   **`RCG_DiminishedPlaceholder`**: replacement placeholder.
*   **`RCG_Extensions.TagColors.EnhenceTitle`**: title color (borrowed from Enhance system).
