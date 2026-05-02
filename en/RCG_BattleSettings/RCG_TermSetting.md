---
title: Term
description: Triggers a Term effect; Terms are managed by the independent RCG_Term system
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Term

> Class: `RCG_TermSetting`

## Purpose
**Triggers a Term effect**. "Terms" are reusable named effects in the game — e.g. **Consume**, **Chant**, **Retain**, **InstantDeath**, **HighSpeedChant** — each with its own trigger logic and tooltip.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Term** | Yes | The Term enum value to trigger (default `HighSpeedChant`). |

## Behaviour
*   On trigger → `RCG_Term.GetTerm(m_Term).TriggerEffect(iData, endAction)`.
*   Description: localized term name (with green title color).
*   **`HasTerm(iTerm)` returns true** when `iTerm == m_Term` — upstream term reverse-lookup finds this setting.
*   `Info` shows the term's full tooltip (unless `Term.Empty`).

## Notes
*   **Term itself decides behaviour**: this setting just **fires a defined Term**. Term logic lives in `RCG_Term.GetTerm(...)`. To add a new Term, see the Term system.
*   **Term.Empty**: legal but inert; useful as a placeholder.
*   **Term trigger timing**: terms like **Retain** / **Consume** have inherent trigger timings (e.g. "on play", "on discard"). Manually firing `TriggerEffect` here is independent — confirm the Term supports active calls.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TermSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_TermSetting` → "Term"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Term` | Term | `Term` (enum) | `Term` | Default `HighSpeedChant` |

### A.3 Key methods
*   **`AddAction`**: `RCG_Term.GetTerm(m_Term).TriggerEffect(iData, iEndAction)`.
*   **`HasTerm(Term iTerm)` (override)** → `iTerm == m_Term`; allows upstream `GetBattleSettings<RCG_TermSetting>` to check term ownership.
*   **`Info`** → null when `Term.Empty`; otherwise `new CardInfoData(RCG_Term.GetTerm(m_Term))`.
*   **`TermDes` (private property)** → `LocalizeName.GetTagColor(TagColors.Term)`.

### A.4 Cross-system interactions
*   **`RCG_Term.GetTerm(Term)`**: term factory; returns logic instance.
*   **`Term` enum**: term ID set (`HighSpeedChant`, `InstantDeath`, `Retain`, `Empty`, ...).
*   **`RCG_Extensions.TagColors.Term`**: term color.

### A.5 Known issues
*   Legacy `Term.Retain` migration is commented out.
