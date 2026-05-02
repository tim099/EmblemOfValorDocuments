---
title: Story Drop Pool (RCG_StoryDropPool)
description: Defines "which stories drop in this context" — source for narrative segments and random story insertion points
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Story Drop Pool

> Class name: `RCG_StoryDropPool`

## Purpose

Defines **which story segments can drop in this context**. Unlike `RCG_QuestDropPool`, this drops `RCG_StoryData` (pure narrative, dialogue, cutscenes), without battles or choices. Examples: "intro played when entering a new chapter", "tale told during nighttime rest".

Inherits from `RCG_Asset<RCG_StoryDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_StoryDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **DropPool** | when DropType=DropPool | Story list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` |

## Behavior

Same skeleton as other Drop Pools; no unlock / skill runtime filtering.

## Caveats

*   Structurally nearly identical to `RCG_QuestDropPool`, differing only in drop target type and tag types.
*   **DropPool mode** auto-cleans invalid IDs on deserialize.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_StoryDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_StoryDropPool>`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_StoryGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropData` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | Default `DropPool` |

### A.3 Key Methods

*   **`GetDrops(int)`** / **`GetDropsWithFilterFunc(int, Func)`** — draw N stories.

### A.4 System Interactions

*   **`RCG_StoryData`** — drop target.
*   **`RCG_StoryGenData`** / **`RCG_StoryDropPoolGenData`** — Asset Entry wrappers.
