---
title: Status Drop Pool (RCG_StatusDropPool)
description: Defines "which status effects drop" — pool used for random buff/debuff and mystery blessing scenarios
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Status Drop Pool

> Class name: `RCG_StatusDropPool`

## Purpose

Defines **which status effects can drop in this context**. Examples: "random blessings from a shrine", "random debuffs from a cursed chest", "group buffs from special events" all draw `RCG_CustomStatusData` from here.

Inherits from `RCG_Asset<RCG_StatusDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_StatusDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← no FilterDrop
    ▼ DropPool / MixDropPools
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` (**only two modes**, no FilterDrop) |
| **DropPool** | when DropType=DropPool | Status list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools |

## Behavior

Similar skeleton to other Drop Pools but **only two modes**: direct list and combine. No "dynamic filter by condition" option.

## Caveats

*   **No FilterDrop mode**: dynamic filtering must be done by manually listing.
*   **Enum is `DropType`** (not `EDropType`).
*   No unlock or other runtime filtering.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StatusDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_StatusDropPool>`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_CustomStatusGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | Only `DropPool` / `MixPool` |

### A.3 System Interactions

*   **`RCG_CustomStatusData`** — drop target.
*   **`RCG_CustomStatusGenData`** / **`RCG_StatusDropPoolGenData`** — Asset Entry wrappers.
