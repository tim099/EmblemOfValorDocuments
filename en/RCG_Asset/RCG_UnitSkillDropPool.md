---
title: Unit Skill Drop Pool (RCG_UnitSkillDropPool)
description: Defines "which unit skills drop on level-up" — pool behind character level-up and skill unlock menus
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Unit Skill Drop Pool

> Class name: `RCG_UnitSkillDropPool`

## Purpose

Defines **which `RCG_UnitSkillData` can drop when leveling up or selecting skills**. For example, the three skill choices offered when a character levels up are drawn from some `RCG_UnitSkillDropPool`.

Inherits from `RCG_Asset<RCG_UnitSkillDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_UnitSkillDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | when DropType=DropPool | Skill list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` |

## Behavior

Same skeleton as other Drop Pools.

## Caveats

*   Enum named `UnitSkillDropType` (no `E` prefix, not just `DropType`) — naming inconsistent with other Drop Pools; historical legacy.
*   Structurally similar to `RCG_CardDropPool`, differing in drop target type (`RCG_UnitSkillData`).

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_UnitSkillDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_UnitSkillDropPool>`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_UnitSkillGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `UnitSkillDropType` enum | Default `DropPool` |

### A.3 Key Methods

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — draw N skills.

### A.4 System Interactions

*   **`RCG_UnitSkillData`** — drop target.
*   **`RCG_UnitSkillGenData`** / **`RCG_UnitSkillDropPoolGenData`** — Asset Entry wrappers.
