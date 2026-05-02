---
title: Card Enhance Drop Pool (RCG_CardEnhenceDropPool)
description: Defines "which enhance branches drop when enhancing a card" — pool behind the enhance selection menu
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card Enhance Drop Pool

> Class name: `RCG_CardEnhenceDropPool`

## Purpose

Defines **which enhance branches (`RCG_CardEnhenceData`) can drop as candidates when enhancing a card**. Each card's `m_EnhencePool` field references a `RCG_CardEnhenceDropPool`; on enhance, the system draws N branches from this pool for the player to choose.

Inherits from `RCG_Asset<RCG_CardEnhenceDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_CardEnhenceDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | when DropType=DropPool | Enhance branch list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` (Operator only) |

## Behavior

Same skeleton as other Drop Pools. The inner `DropFilter` currently **only supports the Operator type** (Tag is commented out), so FilterDrop mode is rarely used in practice — most use DropPool mode with explicit lists.

When drawing, results are also secondary-filtered on the card side by `BannedEnhence` list + `RCG_CardEnhenceCondition` (see `RCG_CardData.GetEnhenceBranchs`).

## Caveats

*   **Enum is `DropType`** (not `EDropType`).
*   **DropFilter Tag is commented out**: FilterDrop mode is functionally limited; recommend DropPool mode with explicit lists.
*   **Enhance condition + BannedEnhence secondary filter** happens on the card side; this pool only handles the initial candidate draw.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardEnhenceDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_CardEnhenceDropPool>`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_CardEnhenceGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | Default `DropPool` |

### A.3 System Interactions

*   **`RCG_CardEnhenceData`** — drop target (enhance branch definition).
*   **`RCG_CardEnhenceGenData`** / **`RCG_CardEnhenceDropPoolGenData`** — Asset Entry wrappers.
*   **`RCG_CardData.m_EnhencePool` / `RCG_CardData.GetEnhenceBranchs`** — call sites that draw from this pool then apply secondary filtering.
