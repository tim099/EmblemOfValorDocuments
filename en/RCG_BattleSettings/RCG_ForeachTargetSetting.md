---
title: Foreach target
description: Run the inner Combine effect once per target (split AOE into a single-target loop)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Foreach target

> Class: `RCG_ForeachTargetSetting`

## Purpose
**Triggers the inner effect once per target** — splits AOE into a "**per-enemy**" loop. Examples:
*   "Insta-kill each enemy below 10 HP" (AOE insta-death must be split per-target to evaluate correctly)
*   "Heal each enemy for 5 HP (with synced animation)"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Targets** | Yes | Target selector (typically AOE). |
| **Content** | Yes | The **Combine effect** to run per target. |

## Behaviour
*   Resolves `Targets` to a list.
*   Per target → creates a child `TriggerEffectData` with `Targets = [target]`, then calls `Content.AddAction`.
*   Description: "**For each {Targets}, {Content}**" (i18n `ForeachTargetDes`).

## Notes
*   **vs Loop**: Loop = "**N times on the same target(s)**"; Foreach = "**Once each on N different targets**".
*   **Preview damage doesn't apply**: this setting **doesn't override `GetPreviewDamage`**; it falls back to base `-1` (non-attack card). Express attack damage in another way upstream.
*   **Nested Foreach**: Foreach inside Foreach is N×M iterations — description becomes very complex. Avoid.
*   **Empty target list**: legal, but nothing happens.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ForeachTargetSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ForeachTargetSetting` → "Foreach target"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Targets` | Targets | `RCG_SelectTargetData` | — | |
| `m_Content` | Content | `RCG_CombineSetting` | — | |

### A.3 Key methods
*   **`AddAction`**: per target in `m_Targets.GetTargets(iData)` → `iData.CreateChildData(target.BattleName)` → `data.Targets = [target]` → `m_Content.AddAction(data, InsertInOrder)`.
*   **Pass-through to `m_Content`**: `Infos`, `HasTerm`, `GetCollaborators`, `GetAtk`.
*   **`GetBattleSettings<T> / (Type)`**: self + recurse `m_Content`.
*   **`GetFusionCandidateSettings`** → delegates to `m_Content`.
*   **`GetFusionBaseSetting`**: clones structure, replaces content with placeholder Combine.

### A.4 Cross-system interactions
*   **`TriggerEffectData.CreateChildData / Targets`**: per-target child data prevents cross-target Targets pollution.
*   **`LocalizedStringUtils.CapitalizeString`**: temporary capitalization hack (shared with LoopSetting).
