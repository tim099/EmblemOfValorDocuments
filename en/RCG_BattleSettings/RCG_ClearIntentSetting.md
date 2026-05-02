---
title: Clear intent
description: Clear the target's next-turn intent (the AI's queued action displayed above their head)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Clear intent

> Class: `RCG_ClearIntentSetting`

## Purpose
**Clear the enemy's next-turn intent** (the icon above their head showing their planned action). Examples:
*   "Cancel the enemy's next attack" mechanics
*   Disrupt / interrupt cards

The enemy will recompute a new intent at the next decision point.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Target** | Yes | Target selector — single or AOE enemies. |

## Behaviour
*   For each target with AI → `target.UnitAI.ClearIntent()`.
*   Description: "**Clear {target}'s intent**" (i18n `ClearIntentDes`).

## Notes
*   **No-AI targets are skipped**: friendly characters typically lack AI; this is ineffective on them.
*   **Doesn't immediately re-decide**: the cleared intent is regenerated at the next AI decision tick — within this turn the enemy won't suddenly attack again.
*   **In-progress intents**: actions already started can't be rolled back; only the unexecuted portion is affected.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ClearIntentSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_ClearIntentSetting` → "Clear intent"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |

### A.3 Key methods
*   **`AddAction`**: per target → check `target.HasAI` → `target.UnitAI.ClearIntent()`.
*   **`GetDescriptionFormat`** → i18n `ClearIntentDes`.

### A.4 Cross-system interactions
*   **`RCG_BattleUnit.HasAI / UnitAI.ClearIntent`**: AI intent clearing entry.
