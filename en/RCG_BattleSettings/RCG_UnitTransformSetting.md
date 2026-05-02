---
title: Unit transform
description: Transform a unit into another type — Default / Reincarnation / Rebirth modes
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unit transform

> Class: `RCG_UnitTransformSetting`

## Purpose
**Transform a unit into another unit type**. Examples:
*   Boss form-change (keep HP, change visuals + behavior)
*   "Reincarnation" / "Rebirth" revival mechanics
*   Special transformation cards

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **Target** | Yes | Transform target selector. |
| **UnitData** | Yes | Unit type to transform into (`RCG_UnitGenData`). |
| **TransformSetting** | Yes | Transform details (nested). |

### TransformSetting fields
| Field | Default | Notes |
|---|---|---|
| **TransformType** | `Default` | Mode:<br>• **Default** — generic. **Keeps HP and statuses**, changes only behavior + model (**Boss form-change**).<br>• **Reincarnation** — keeps **only statuses**; HP/cap/stats become the new unit's; triggers entrance actions.<br>• **Rebirth** — drops statuses; HP/cap/stats become the new unit's; triggers entrance actions. |
| **ClearAllStatus** | `false` | Clear all statuses (independent of TransformType's "keep statuses" logic). |

## Behaviour
*   Description: `UnitTransformDes_{TransformType}`, with target and unit name interpolated.
*   At runtime, per TransformType decides which props to keep, then performs visual swap (Spine model / AI / behavior).

## Notes
*   **Default vs Reincarnation**: Default keeps all current values (Boss form-change); Reincarnation resets HP but keeps statuses.
*   **Entrance actions fire**: Reincarnation / Rebirth call the new unit's `OnBattleStart` (entrance buffs / attacks).
*   **ClearAllStatus + TransformType conflict**: e.g. Default + ClearAllStatus = "keep HP but drop statuses" — uncommon combo, use carefully.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_UnitTransformSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_UnitTransformSetting` → "Unit transform"
*   **Top-level type**: `TransformSetting` (with `TransformType` enum + `m_ClearAllStatus`)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_Target` | Target | `RCG_SelectTargetData` | `Target` | |
| `m_UnitData` | UnitData | `RCG_UnitGenData` | `UnitData` | |
| `m_TransformSetting` | TransformSetting | `TransformSetting` | — | With `TransformType` and `ClearAllStatus` |

### A.3 Key methods
*   **`AddAction`** (beyond shown range): branches on `TransformType` to perform actual transform.
*   **`GetDescriptionFormat`** → `UnitTransformDes_{TransformType}` i18n key.
*   **`Info`** → `new CardInfoData(m_UnitData.GetData())`.

### A.4 Cross-system interactions
*   **`RCG_Unit` transform entry** (specific method beyond shown range).
*   **`RCG_UnitGenData`**: target unit template.
*   **`OnBattleStart`**: Reincarnation / Rebirth entrance trigger.
