---
title: Trigger intent
description: Force-fire the target's next-turn action (move attack forward / mimic enemy skill)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Trigger intent

> Class: `RCG_TriggerIntentSetting`

## Purpose
**Force-fire the target's queued next-turn intent** (the icon over their head). Examples:
*   "Force the enemy to attack early" (player can preempt)
*   "Mimic enemy skill" (copy their action)
*   "Trigger and refresh" (fire then re-roll)

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **TriggerIntentType** | Yes | Mode:<br>• **Default** — fire intent<br>• **TriggerAndRefresh** — fire then refresh<br>• **ImitateIntention** — mimic the target's intent (the user executes the target's queued action) |
| **Target** | Yes | Target unit selector; non-AI targets are filtered out. |

## Behaviour
*   Resolves `Target`, filters by `HasAI`, sorts by `BattleID` for stability.
*   **Default / TriggerAndRefresh**: per target enqueue `RCG_IntentAction(target, type)` at `AddActionMode.Last` (end of queue).
*   **ImitateIntention**: take the **first** target's `UnitAI.CurrentAction.m_UnitAction` and run it as the user — mimics their attack.

## Notes
*   **No-AI targets skipped**: friendly units typically have no AI; this is ineffective on allies.
*   **ImitateIntention picks one target**: multi-target selectors only use the first.
*   **vs Clear intent**: Clear lets the enemy re-decide; this **forces execution of the current intent**.
*   **`Last` insertion mode**: intent fires **after other actions complete** — different from `InsertInOrder`. Mind the chaining order.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_TriggerIntentSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_TriggerIntentSetting` → "Trigger intent"
*   **Top-level enum**: `ETriggerIntentType` (`Default`, `TriggerAndRefresh`, `ImitateIntention`)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_TriggerIntentType` | TriggerIntentType | `ETriggerIntentType` | — | 3 modes |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |

### A.3 Key methods
*   **`AddAction`**: `m_Target.GetTargets(iData).Where(HasAI).OrderBy(BattleID)`, branches per `m_TriggerIntentType`:
    *   `ImitateIntention` → take `aTargets.FirstOrDefault().UnitAI.CurrentAction.m_UnitAction.AddAction(iData, InsertInOrder)`.
    *   Others → per target `iData.AddAction(new RCG_IntentAction(target, m_TriggerIntentType), AddActionMode.Last)`.
*   **`GetDescriptionFormat`** → `m_TriggerIntentType.GetLocalizeDes(target)`.

### A.4 Cross-system interactions
*   **`RCG_BattleUnit.HasAI / UnitAI.CurrentAction`**: AI current-action query.
*   **`RCG_IntentAction`**: actual Action class.
*   **`AddActionMode.Last`**: end-of-queue insertion (≠ `InsertInOrder`).
