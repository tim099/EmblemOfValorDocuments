---
title: End battle
description: Immediately ends the current battle with a specified WinState (flee, win, lose, etc.)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# End battle

> Class: `RCG_BattleEndSetting`

## Purpose
**Forcibly ends the battle**. The most common use is **Flee** cards that let the player leave combat.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **WinState** | Yes | Outcome: `Flee` / `Win` / `Lose` / etc. (see `WinState` enum). |

## Behaviour
*   **Playability**: only allowed when the enemy type permits flight (`m_EnemyType.GetData().m_CanFlee`). Boss fights typically set `m_CanFlee = false`.
*   **Trigger**: queues `RCG_BattleEndAction(WinState)`; battle resolution proceeds per WinState.
*   Description: shows the `WinState` localized name (e.g. "Flee", "Victory").

## Notes
*   **Boss / forced battles**: don't accidentally let players Flee out — set `EnemyType.m_CanFlee = false`.
*   **Win / Lose use sparingly**: outcomes are normally HP-driven; only special story logic (auto-fail, forced victory) should use these manually.
*   **Card greys out without explanation**: if `CanFlee` is false, the card disables but the description still says "Flee" — players may be confused. Add a card note.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_BattleEndSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_BattleEndSetting` → "End battle"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_WinState` | WinState | `WinState` (enum) | — | Default `Flee` |

### A.3 Key methods
*   **`CheckPlayable`** → `CanFlee`, queries `RCG_BattleManager.Ins.m_BattleData.m_EnemyType.GetData().m_CanFlee`; defaults to `true` when no manager.
*   **`GetDescriptionFormat / GetDescriptionShort`** → `m_WinState.GetLocalizeDes()`.
*   **`AddAction`** → `iData.AddAction(new RCG_BattleEndAction(m_WinState), iAddActionMode)` only if `CanFlee` (double-checked).

### A.4 Cross-system interactions
*   **`RCG_BattleEndAction`**: actual resolution Action.
*   **`RCG_EnemyType.m_CanFlee`**: per-enemy-type flee permission.
