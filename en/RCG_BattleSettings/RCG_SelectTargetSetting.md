---
title: Select target
description: Reselect targets via player click; overrides the existing target list
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Select target

> Class: `RCG_SelectTargetSetting`

## Purpose
**Lets the player re-pick attack / effect targets** (chooser UI). Downstream settings then use the new target list. Examples:
*   "First select a target, then deal 5x damage to it"
*   Complex effects requiring manual targeting

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **TargetType** | Yes | Selectable target type (`TargetType` enum, e.g. All / Enemy / Ally / SingleEnemy). |

## Behaviour
*   On trigger: unblocks battle input → opens target chooser → player click fills `iData.Targets` → re-blocks.
*   Description: localized name of `TargetType`.
*   Battle pauses during selection until the player clicks.

## Notes
*   **Invalid for AI**: relies on player input; AI triggering this hangs the battle. **AI should use SetTarget** instead.
*   **Overrides original targets**: post-selection `iData.Targets` is overwritten; subsequent settings see the new ones.
*   **vs SetTarget**: this version **shows UI**; SetTarget **applies a rule automatically**.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectTargetSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_SelectTargetSetting` → "Select target"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_TargetType` | TargetType | `TargetType` (enum) | — | Default `All` |

### A.3 Key methods
*   **`AddAction`** (async): `RCG_BattleManager.Ins.UnBlockAction(true)` → `await RCG_BattleField.Ins.SelectTargetsAsync(m_TargetType, false, iToken)` → `iData.SetTargets(targets)` → `BlockAction()`.
*   **`GetDescriptionFormat`**: directly displays `m_TargetType.GetDescription()`.

### A.4 Cross-system interactions
*   **`RCG_BattleField.Ins.SelectTargetsAsync`**: target chooser entry.
*   **`RCG_BattleManager.UnBlockAction / BlockAction`**: input lock control.
*   **`TargetType` enum**: selectable type set.
