---
title: Quest Drop Pool (RCG_QuestDropPool)
description: Defines "which quests trigger in this context" — source for map nodes and random events
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Quest Drop Pool

> Class name: `RCG_QuestDropPool`

## Purpose

Defines **which quests can trigger in this context**. For example, "medium town" / "forest mystery node" / "night encounter" are different pools, each listing possible `RCG_QuestData` (event scripts: dialogue, choices, results) + weights.

Inherits from `RCG_Asset<RCG_QuestDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_QuestDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | when DropType=DropPool | Quest list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools with weights |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` (FilterType: Tag / Operator) |

## Behavior

### Three Modes
Same as other Drop Pools. FilterDrop supports only one dimension: "event tag".

### Implicit Filtering
This class **has no unlock filtering** — the default filter always returns true. There's commented-out logic for "don't trigger the same event consecutively" (`triggeredStory.LastElement()`) currently disabled.

## Caveats

*   **Enum is `DropType`, not `EDropType`**: inconsistent with BattleSetDropPool / ItemDropPool naming — historical legacy.
*   **No Name field**: `GetShortName()` returns the first drop's name directly.
*   **No `DeserializeFromJson` override** for cleaning invalid IDs — bad IDs persist; need manual cleanup.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_QuestDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_QuestDropPool>`
*   **Implements**: `UCL.Core.UCLI_ShortName`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_QuestGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropDataBase<RCG_QuestGenData, RCG_QuestData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `DropType` enum | Default `DropPool` |

### A.3 Key Methods

*   **`GetDrops(int, CheckDropConditionData)`** — main entry.
*   **`GetDropsWithFilterFunc(int, Func)`** — custom filter version.
*   **`GetDropRate(CheckDropConditionData, int)`** — default filter always true.

### A.4 System Interactions

*   **`RCG_QuestData`** — drop target type.
*   **`RCG_QuestGenData`** / **`RCG_QuestDropPoolGenData`** — Asset Entry wrappers; latter has `DefaultID = "NormalDrop"`.
*   **`RCG_EventTagGenData`** — tag type for FilterDrop.

### A.5 Known Issues

*   Commented-out "don't trigger same event consecutively" logic — uncomment to restore.
*   No `DeserializeFromJson` override; bad IDs aren't cleaned automatically.
