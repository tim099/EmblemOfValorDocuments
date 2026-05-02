---
title: Capture unit
description: Removes the target unit from battle and creates a summon item in the player's inventory
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Capture unit

> Class: `RCG_CaptureUnitSetting`

## Purpose
**Captures** the target unit — removes it from the battlefield and creates a "**summon item**" in the player's inventory. Pokémon-style capture: post-battle, the player can deploy the captured creature as an ally.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Target** | Yes | Unit to capture (typically single enemy). |
| **CaptureUnitItem** | Yes | Item template for the capture result; defaults to `Vivarium_Captured`. |

## Behaviour
*   On trigger:
    1. Plays capture VFX (`VFX_Summon`) + shrink animation (0.6s) on the target.
    2. The target is `Flee()`'d (removed from battlefield, **not counted as a kill**).
    3. Clones `CaptureUnitItem` into a new `RCG_ItemData`; renames to `{itemName}[{capturedUnit}]`.
    4. New item ID becomes `{originalID}_{unitID}` to avoid collisions.
    5. Item's "use effect" is auto-wired to "summon (the captured unit)".
    6. Item added to inventory + acquire panel popup.
*   Description: "**Capture {target}**" (i18n `CaptureUnitDes`).

## Notes
*   **Capture isn't a kill**: uses `Flee()` to remove, so kill-based conditions/counters **won't fire**.
*   **Multi-target selectors**: only the **first** target is captured (`aTargets[0]`); pick a single-target selector.
*   **CaptureUnitItem must exist**: pointing at an unregistered ID throws on item creation.
*   **Naming collisions**: capturing the same monster twice produces same-ID items; second overrides first (known behavior).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CaptureUnitSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CaptureUnitSetting` → "Capture unit"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |
| `m_CaptureUnitItem` | CaptureUnitItem | `RCG_ItemGenData` | — | Default `RCG_ItemGenData("Vivarium_Captured")` |

### A.3 Key methods
*   **`AddAction`** (async):
    1. `m_Target.GetTargets(iData)[0]` → `aTarget`.
    2. `RCG_VFXManager.CreateVFX(CommonVFX.VFX_Summon)` at target.
    3. `aTarget.UnitAnimController.ScaleUnitLocal(...)` shrink anim.
    4. `aTarget.Flee()` removes from field.
    5. Clone `m_CaptureUnitItem.GetData()` → modify name (`LocalizeDic`) + ID suffix (unitID).
    6. Wire `RCG_CommonEffect(OnPlay) → RCG_SummonSetting(SummonType.Default, IsMonster=false, Unit=capturedMonster)`.
    7. `RCG_DataService.Ins.AddRuntimeData(aItemData, DataType.InGameRuntime)`.
    8. New `RCG_Item(aItemData, ItemType.Runtime).AddItem()`.
    9. Show via `RCG_AquireItemPanel`.
*   **`GetDescriptionFormat`** → i18n `CaptureUnitDes`, single `{Target}` param.

### A.4 Cross-system interactions
*   **`RCG_VFXManager / CommonVFX.VFX_Summon`**: capture VFX.
*   **`RCG_BattleUnit.Flee`**: non-kill removal entry.
*   **`RCG_DataService / DataType.InGameRuntime`**: runtime item registration.
*   **`RCG_SummonSetting`**: built into the item's use-effect.
*   **`RCG_AquireItemPanel`**: UI feedback.
