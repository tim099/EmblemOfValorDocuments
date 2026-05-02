---
title: Monster move
description: Moves a monster on the battlefield (typically front/back swap); used by AI behavior
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Monster move

> Class: `RCG_MonsterMoveSetting`

## Purpose
**Moves a monster on the battlefield** — usually a front/back row swap. Used by monster AI behavior. Examples:
*   Enemy retreats to the back row to dodge
*   Special unit positioning skills

> [!IMPORTANT]
> Only appears in the dropdown when **editing monster data**. Not visible on regular cards.

## Key fields
(no fields — pure behavior)

## Behaviour
*   Calls `CreateAction.MoveAction(iData.User)` to move the unit.
*   Actual move logic is decided by `MoveAction` internals (typically front/back swap).
*   Description: i18n `UnitAction_MoveActionDes`; short description shows the move sprite.

## Notes
*   **vs Move setting**: **Move** (`RCG_MoveSetting`) is the **general-purpose** version with direction + target params; this is **monster AI specific** with no fields.
*   **No fields = fixed behavior**: move logic is set by `CreateAction.MoveAction`. For custom behavior use **Move**.
*   **No-User case**: silently skips when `iData.User == null`.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MonsterBattleSettings/RCG_MonsterMoveSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **No i18n class key**: editor shows stripped name `MonsterMove`
*   **Restricted dropdown visibility**: registered in `RCG_BattleSetting.s_MonsterDataTypes`.

### A.2 Field map
(no fields)

### A.3 Key methods
*   **`AddAction`**: when `iData.User != null` → `iData.AddAction(CreateAction.MoveAction(iData.User), iAddActionMode)`.
*   **`GetDescription`** → i18n `UnitAction_MoveActionDes`.
*   **`GetDescriptionShort`** → move sprite (`EffectIcon.Move`).

### A.4 Cross-system interactions
*   **`CreateAction.MoveAction(unit)`**: actual Action builder (shared with general Move setting? or monster-AI specific).
