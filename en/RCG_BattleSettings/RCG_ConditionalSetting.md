---
title: Conditional
description: Branches by AND-conditions; runs IfConditionFit on success, ElseCondition on failure
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Conditional

> Class: `RCG_ConditionalSetting`

## Purpose
Branches based on **CardConditions** (AND-combined). On success runs **IfConditionFit**; otherwise runs **ElseCondition**. Examples:
*   "If kill: gain 1 Agile, else deal 3 extra damage"
*   "If HP < 50%: heal self, else gain armor"
*   "If hand size ≥ 5: draw 1, else discard 1"

## Inspector layout
```
▼ ✓ [Conditional(Conditional)] If (conditions) then ... else ...
    CardConditions       ▶ (condition list)
    IfConditionFit       ▶ (child setting list)
    ElseCondition        ▶ (child setting list)
```

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CardConditions** | Usually | Condition list, AND-combined. **All** must pass to count as "fit". Empty list = always true. |
| **IfConditionFit** | Yes (≥1) | Children to run when conditions are met. |
| **ElseCondition** | No | Children to run when conditions fail. May be empty. |

## Behaviour

### Condition evaluation
*   AND across all conditions; one failure → overall false.
*   Empty list = **always true** (always takes the IfConditionFit branch).
*   Each condition has its own config (e.g. "on kill", "HP ratio", "has status"). See per-condition docs.

### Runtime
1. Trigger evaluates conditions.
2. Fit → run all enabled children of IfConditionFit.
3. Not fit → run all enabled children of ElseCondition.
4. Disabled children (unchecked) are skipped.

### Description
*   With conditions + If: "**If {conditions}, then {fit content}**"
*   With Else: extra newline + "**else {else content}**"
*   Multiple conditions joined by "and"

### Preview damage limitation
The preview pre-evaluates conditions to pick a branch and takes max damage.

> [!WARNING]
> For **outcome conditions** (e.g. "if kill", "if crit"), the preview can't know the future, so the displayed value may differ from actuality. **For accurate previews, use stable conditions** (HP ratio, stack count, owned status).

## Notes

*   **Empty CardConditions + non-empty Else** = anti-pattern: condition always passes, Else never runs but still shows in description. **Add a condition or remove Else**.
*   **Avoid deep nesting**: "If A then: if B then: ..." is unreadable. **Use multi-condition AND** instead.
*   **Conditions are AND, not OR**: For OR, use two separate Conditionals (or extend the engine).
*   **Disabled children**: still appear in description aggregation in some paths — keep data clean.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ConditionalSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ConditionalSetting` → "Conditional"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CardConditions` | CardConditions | `List<RCG_Condition>` | `CardConditions` | AND-combined; empty = true |
| `m_IfConditionFit` | IfConditionFit | `List<RCG_BattleSetting>` | `IfConditionFit` | |
| `m_ElseCondition` | ElseCondition | `List<RCG_BattleSetting>` | `ElseCondition` | |
| `IfConditionFit` (prop) | — | `List<RCG_BattleSetting>` | — | `m_IfConditionFit.GetEnableBattleSettings()` |
| `ElseCondition` (prop) | — | `List<RCG_BattleSetting>` | — | `m_ElseCondition.GetEnableBattleSettings()` |

### A.3 Key methods
*   **`AddAction`**: in `AddActionTrigger`, `m_CardConditions.CheckConditions_AND(iData)` → branches; each child `AddAction(iData, InsertInOrder)`.
*   **`Infos`**: aggregates `m_CardConditions[*].Infos + IfConditionFit[*].Infos + ElseCondition[*].Infos` with dedup.
*   **`GetBattleSettings<T> / (Type)`**: self + recurse **both branches** (not branch-picking).
*   **`GetDescriptionFormat`**: temporarily disables `m_FullSentence`, builds with `IfCardConditions` + `ConditionFitThenDes` + optional `ElseTriggerEffectsDes`, then restores.
*   **`ConditionDes` (private)**: joins `m_CardConditions[*].Description` with `WordSeperator + ConditionAnd + WordSeperator`.
*   **`GetPreviewDamage`**: branch-picks via current `iData`, then `Mathf.Max` over chosen branch (inaccurate for outcome conditions).
*   **`PreloadData`**: `await` both branches.
*   **`GetFusionCandidateSettings`**: aggregates enabled children's candidates from both branches (not self).
*   **`GetFusionBaseSetting`**: clones structure, rebuilds branches as placeholders, drops disabled.

### A.4 Cross-system interactions
*   **`RCG_Condition`**: condition base; provides `Description` / `CheckCondition(iData)`.
*   **`CheckConditions_AND` (extension)**: returns true only if all pass.
*   **i18n keys**: `IfCardConditions`, `ConditionFitThenDes`, `ElseTriggerEffectsDes`, `ConditionAnd`.
*   **`LocalizedStringUtils.WordSeperator()`**: locale-aware separator (CJK no space, EN with space).

### A.5 Known issues
*   Legacy ElseTriggerEffects logic kept as comments for reference.
*   `m_FullSentence` save/restore is manual — vulnerable to throwing exceptions; `try/finally` would be safer.
*   Preview damage inaccuracy on outcome conditions is a design tradeoff.
