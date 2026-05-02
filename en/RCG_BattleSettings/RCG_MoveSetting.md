---
title: Move
description: Move units between positions (front/back rows, opposite, or specified positions)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Move

> Class: `RCG_MoveSetting`

## Purpose
**Move units to designated positions**. Examples:
*   "Position swap" cards — pull back-row to front
*   "Charge" — move to enemy position
*   "Retreat" — move to back row

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **MoveType** | Yes | Mode:<br>• **ToOther** — swap to opposite row (default)<br>• **ToFront** — move to front<br>• **ToBack** — move to back<br>• **ToTarget** — move to a specified position (paired with `TargetPositions`) |
| **MoveTarget** | Yes | Selector for the units to move. |

## Behaviour
*   `ToFront` / `ToBack` / `ToOther`: per target moves to the corresponding row:
    *   `ToOther` → swap front↔back
    *   `ToFront` → always front
    *   `ToBack` → always back
*   `ToTarget`: pairs by index with `TargetPositions` (up to N units to N positions).
*   Description: "**{MoveTarget} moves {MoveType}**" (i18n `Move_Des`).

## Notes
*   **Dead targets skipped**: per-iteration `IsNullOrDead` check.
*   **ToTarget needs `TargetPositions`**: typically preceded by **SetTarget** or similar to populate `iData.m_TriggerData.m_TargetPositions`; otherwise nowhere to go.
*   **Position occupied**: `RCG_BattleField.MoveUnit` decides — usually pushes / swaps.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_MoveSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_MoveSetting` → "Move"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_MoveType` | MoveType | `UnitMoveType` (file enum) | `MoveType` | 4 modes |
| `m_MoveTarget` | MoveTarget | `RCG_SelectTargetData` | — | |

### A.3 Key methods
*   **`AddAction`**: branches on `m_MoveType`:
    *   `ToTarget` → pair `m_TargetPositions` and `aTargets` by index, call `RCG_BattleField.Ins.MoveUnit(target, position, token)`.
    *   Others → per target compute target row, same `MoveUnit` call.
*   **`GetDescriptionShort`** → move sprite (`EffectIcon.Move`).

### A.4 Cross-system interactions
*   **`RCG_BattleField.Ins.MoveUnit(unit, pos, token)`**: actual move entry (with anim).
*   **`UnitPos`**: front / back enum.
*   **`iData.m_TriggerData.m_TargetPositions`**: source positions for `ToTarget`.
