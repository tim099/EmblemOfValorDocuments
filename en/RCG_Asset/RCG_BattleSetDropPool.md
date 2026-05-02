---
title: Battle Set Drop Pool (RCG_BattleSetDropPool)
description: Defines "which battle setups appear at this map node" — random pool behind map events and chapter encounters
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Set Drop Pool

> Class name: `RCG_BattleSetDropPool`

## Purpose

Defines **which battle setups can appear when stepping on a particular battle node**. For example, "normal encounter" / "elite encounter" / "boss fight" are different pools; each pool lists possible `RCG_BattleSet` (each BattleSet is a specific monster configuration) + weights.

Inherits from `RCG_Asset<RCG_BattleSetDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_BattleSetDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | when DropType=DropPool | BattleSet list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools with weights |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` (Tag / EnemyType / Operator) |

## Behavior

### Three Modes
Same as other Drop Pools. FilterDrop supports two dimensions: "BattleSet tag" and "Enemy type".

### Implicit Filtering
This class **has no unlock / skill runtime filtering** — picks straight by weight.

## Caveats

*   **The enum is `EDropType`, not `DropType`**: a class-level comment notes "naming it DropType breaks GoogleSheet language file sync, renamed". Don't try to rename it back.
*   **DropPool mode** auto-removes invalid BattleSet IDs on deserialize.
*   **MixPool cycles** are cut off as empty (`iLayer > 10`).
*   **No Name field**; `GetShortName()` returns the first drop's name directly.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_BattleSetDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleSetDropPool>`
*   **AssetGroup**: `EditBattleSetting` (note: differs from other DropPools which are in `EditDropSetting`)

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_BattleSetGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropDataBase<RCG_BattleSetGenData, RCG_BattleSet, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | Default `DropPool` |

### A.3 Key Methods

*   **`GetBattleSets(int)`** — main entry; no built-in filter, draws N directly.
*   **`GetBattleSetsWithFilterFunc(int, Func)`** — custom filter version.
*   **`GetDropRate(CheckDropConditionData, int)`** — default filter is `_ => true`.

### A.4 System Interactions

*   **`RCG_BattleSet`** — drop target type.
*   **`RCG_BattleSetGenData`** / **`RCG_BattleSetDropPoolGenData`** — Asset Entry wrappers; latter has `DefaultID = "NormalBattle"`.
*   **`RCG_BattleSetTagGenData`** / **`RCG_EnemyTypeTagGenData`** — tag types for FilterDrop.

### A.5 Known Issues

*   The `enum EDropType` (E prefix) is a historical workaround for GoogleSheet sync conflicts.
*   Recursion cap at 10 layers.
