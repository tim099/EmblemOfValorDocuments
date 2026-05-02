---
title: Change party order
description: Reorder party members (rotate first to last, or pull a target to the front)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Change party order

> Class: `RCG_ChangePartyOrderSetting`

## Purpose
**Rearrange party members' battle order**. Examples:
*   "Switch character" cards (rotate first to back)
*   "Pull-front" effects (move targeted character to first slot)

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **ChangePartyOrderType** | Yes | Mode:<br>• **FirstToLast** — rotate first slot to the back<br>• **TargetToFirst** — move chosen character to the front |
| **Target** | Yes | Target selector (must be single for `TargetToFirst`). |

## Behaviour
*   `FirstToLast`: directly calls `RCG_BattleManager.Ins.SwitchCharacterUI.FirstToLast()`.
*   `TargetToFirst`: for a single-target → plays "light point" VFX, then switches UI.
*   Description per type: `ChangePartyOrderDes_FirstToLast` / `ChangePartyOrderDes_TargetToFirst`.

## Notes
*   **TargetToFirst with multi-target**: only single-target works as designed (code path checks `aTargets.Count == 1` before VFX).
*   **Pacing impact**: changing order shifts the next acting unit; verify downstream triggers depend on the right "current actor".
*   **Always expanded**: `[AlwaysExpendOnGUI]`.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_ChangePartyOrderSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable] + [AlwaysExpendOnGUI]`
*   **i18n class key**: `RCG_ChangePartyOrderSetting` → "Change party order"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_ChangePartyOrderType` | ChangePartyOrderType | `ChangePartyOrderType` (file enum) | — | `FirstToLast` / `TargetToFirst` |
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | Replaces deprecated `m_Range` |

### A.3 Key methods
*   **`AddAction`** (async): branches on type. `TargetToFirst` plays `VFX_LightTarget` only when `aTargets.Count == 1`.
*   **`GetDescriptionShort`** → returns i18n key `RCG_ChangePartyOrderSetting` literal label.

### A.4 Cross-system interactions
*   **`RCG_BattleManager.Ins.SwitchCharacterUI`**: actual reorder UI / logic.
*   **`CommonVFX.VFX_LightTarget`**: pull-front VFX.
