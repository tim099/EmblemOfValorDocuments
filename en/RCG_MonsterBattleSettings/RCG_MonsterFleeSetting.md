---
title: Monster flee
description: Lets a monster flee the battlefield with a horizontal move animation, then removes them
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Monster flee

> Class: `RCG_MonsterFleeSetting`

## Purpose
**Makes a monster flee the battlefield** — plays a sideways move-out animation, then removes the unit (**not counted as a kill**, doesn't trigger kill-based effects). Examples:
*   Per-monster "flee" behavior (e.g. when low HP)
*   Story-driven "enemy retreats" cutscenes

> [!IMPORTANT]
> Only appears in the dropdown when **editing monster data** (`RCG_UnitData` / `RCG_MonsterActionData`). Not visible on regular cards.

## Key fields
(no fields — pure behavior)

## Behaviour
*   Plays a horizontal move animation: enemy units move +2000 right, player units -2000 left (0.8s).
*   After animation → `aUser.Flee()` removes from battlefield.
*   Description: i18n `FleeActionDescription` (full) / `FleeActionDescriptionShort` (short).

## Notes
*   **Not a kill**: distinct from "InstantDeath" / "death"; **kill counters and on-kill effects don't fire**.
*   **No fields = fixed behavior**: direction and duration are hardcoded; for custom behavior compose other settings.
*   **Monster-only by intent**: technically works on player units too, but the logic is designed for "enemy flees".

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterFleeSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **No i18n class key**: editor shows stripped name `MonsterFlee`
*   **Restricted dropdown visibility**: registered in `RCG_BattleSetting.s_MonsterDataTypes` (only appears when editing monster data).

### A.2 Field map
(no fields)

### A.3 Key methods
*   **`AddAction`** (async): per `iData.User.UnitFaction` decides `aX = -2000` or `+2000`, calls `UnitAnimController.MoveUnitLocal(token, new Vector3(aX, 0), 0.8f, false)`, then `aUser.Flee()`.
*   **`GetDescription / GetDescriptionShort`**: i18n `FleeActionDescription` / `FleeActionDescriptionShort`.

### A.4 Cross-system interactions
*   **`RCG_BattleUnit.Flee`**: removal entry (shared with Capture).
*   **`UnitAnimController.MoveUnitLocal`**: sideways animation.
