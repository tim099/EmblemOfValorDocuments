---
title: Player energy alter
description: Modify the player's current energy / cost — add, subtract, or set to value
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Player energy alter

> Class: `RCG_CostAlterSetting`

## Purpose
**Change the player's current energy (cost)**. Examples:
*   "Gain 1 energy"
*   "Spend 2 energy for a powerful effect"
*   "Set energy to 0" (overload)

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CostAlter** | Yes | Delta (variable). |
| **CostAlterType** | Yes | Mode:<br>• **Add** — gain<br>• **Sub** — spend (**gates playability**)<br>• **Set** — set to value |

## Behaviour
*   **Playability**: `Sub` mode + literal value → current energy must be ≥ `CostAlter` (avoids negative). Other modes don't check.
*   **Trigger**: per mode, calls `RCG_Player.Ins.AlterCost(...)` or `SetCost(...)`.
*   **VFX**: gain plays `ChargeEffect`; spend plays `OverloadEffect`; both show floating energy numbers.
*   Description per mode: i18n `AddCostIcon_Des` / `SubCostIcon_Des` / `SetCostIcon_Des` + energy icon.

### Card fusion
Same `CostAlterType` cards fuse by adding `CostAlter`.

## Notes
*   **Legacy data migration**: previously, "spend energy" used a negative `CostAlter`; now uses `CostAlterType.Sub`. `DeserializeFromJson` auto-flips negatives to `Sub` + positive value.
*   **Variable Sub doesn't gate playability**: only `m_VariableType == Value` mode gates; variable-driven Sub may briefly produce negative energy.
*   **Set isn't gated**: always playable; "Set 0" cards lose tactical play space — design carefully.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CostAlterSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CostAlterSetting` → "Player energy alter"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CostAlter` | CostAlter | `IntVariable` | — | |
| `m_CostAlterType` | CostAlterType | enum (file) | — | `Add` / `Sub` / `Set` |

### A.3 Key methods
*   **`CheckPlayable`**: `Sub` + literal → `RCG_Player.Ins.Cost >= m_CostAlter`; otherwise true.
*   **`DeserializeFromJson`**: legacy negative CostAlter → flip to `Sub` + positive (mutates `m_Value` or `m_Mult`).
*   **`AddAction`** (async): per mode → `RCG_Player.Ins.AlterCost(value, isCardCost: false)` or `SetCost(value)`; plays `ChargeEffect` / `OverloadEffect` + `VFX_Cost`.
*   **`Fusion`**: requires same type; clone + `IntVariable.FuseAdd`.

### A.4 Cross-system interactions
*   **`RCG_Player.Ins.AlterCost / SetCost`**: energy mutation entry.
*   **`RCG_CommonVFXGenData.s_ChargeEffect / s_OverloadEffect`**: VFX.
*   **`CommonVFX.VFX_Cost`** (via `aCostVFX.SetAlterHP`): floating energy UI.
