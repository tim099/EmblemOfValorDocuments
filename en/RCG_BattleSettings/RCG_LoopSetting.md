---
title: Loop trigger
description: Repeats the inner Combine effect N times; classic "deal X damage N times" pattern
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Loop trigger

> Class: `RCG_LoopSetting`

## Purpose
Repeats a **Combine effect** N times. Examples:
*   "Deal 1 damage, repeat 5 times" (multi-hit)
*   "Each remaining hand card triggers once" (variable-bound)
*   "Grant 1 Agile, repeat 3 times" (status stacking)

## Inspector layout
```
▼ ✓ [Loop trigger(Loop)] (LoopContent) × N
    LoopTimes      [Value] 1
    LoopContent    ▶ (Combine effect container)
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **LoopTimes** | Yes | Repeat count; supports variables (e.g. "= remaining cards"). |
| **LoopContent** | Yes | A **Combine effect** holding the repeated content. |

> [!NOTE]
> Why is the content a Combine instead of a `List<RCG_BattleSetting>`? Combine already handles description aggregation, tag dedup, and preview rollup — wrapping saves work. **Single-effect loops also use a one-child Combine**.

## Behaviour

### What happens in battle
1. Resolve **LoopTimes** to N.
2. Run LoopContent N times in sequence; each iteration's actions are inserted into the queue in order.

### Description
"Repeat N times: {content}" (auto-capitalized first letter).
Short form (enemy intent): `{content} × {N}`.

### Preview damage doesn't multiply by N
The card's preview shows **per-iteration** damage, not totals. Example: "Repeat 3 times, deal 4 damage" shows 4, NOT 12.

> [!IMPORTANT]
> Design choice — players read "per-hit". To emphasize totals, write "**3 × 4 = 12**" in card text.

## Notes

*   **LoopTimes ≤ 0**: doesn't crash but produces no effect — clamp to `Mathf.Max(1, ...)` or design around it.
*   **Loop inside Loop**: technically valid, produces "Repeat M times: Repeat N times: ..." which is awful UX. Flatten to `M*N`.
*   **Multi-target + Loop**: "AOE attack + Loop 3" ≠ "each enemy hit 3 times". Former is "3 AOE waves with re-target each iteration"; latter needs **Foreach target**.
*   **AOE inside Loop**: re-resolves targets each iteration, so dead enemies are skipped going forward (intuitive).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_LoopSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_LoopSetting` → "Loop trigger"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_LoopTimes` | LoopTimes | `IntVariable` | `LoopTimes` | |
| `m_LoopContent` | LoopContent | `RCG_CombineSetting` | `LoopContent` | Wrapping for aggregation reuse |

### A.3 Key methods
*   **`AddAction`**: `for i in [0, LoopTimes): m_LoopContent.AddAction(iData, InsertInOrder)`. Critical: `InsertInOrder` preserves sequencing.
*   **`GetDescriptionFormat`**: temporarily sets `m_FullSentence = false`, capitalizes content, then `LoopSettingDes` template.
*   **`GetDescriptionShort`** → `LoopSettingDesShort`: `{content} × {times}`.
*   **Pass-through to `m_LoopContent`**: `Infos`, `HasTerm`, `GetCollaborators`, `GetAtk`, `GetPreviewDamage`, `PreloadData`.
*   **`GetBattleSettings<T> / (Type)`** → self + recurse content.
*   **`GetFusionCandidateSettings`** → delegates to content (Loop is structural, not a leaf).
*   **`GetFusionBaseSetting`**: clones structure, replaces content with placeholder Combine.

### A.4 Cross-system interactions
*   **`RCG_CombineSetting`**: container reused for description aggregation.
*   **`AddActionMode.InsertInOrder`**: ensures iteration ordering.
*   **`IntVariable.GetValue`**: not clamped, so 0/negative produce zero iterations.

### A.5 Known issues
*   `LocalizedStringUtils.CapitalizeString` is marked as a temporary hack in source; a proper i18n solution would replace it.
*   Preview damage not multiplying by N is intentional, not a bug.
*   Legacy `CanEnhence` / `Enhence` commented out.
