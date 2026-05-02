---
title: Item Drop Pool (RCG_ItemDropPool)
description: Defines "which items drop, with what weights" — event rewards, treasure chests, shop restocks all draw items through this
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Item Drop Pool

> Class name: `RCG_ItemDropPool`

## Purpose

Defines **which consumable items this pool drops and the weight of each**. Treasure chests, event rewards, merchant restocks draw items here (potions, scrolls, ingredients, etc.). Same skeleton as `RCG_CardDropPool`, just dropping `RCG_ItemData` instead of cards.

Inherits from `RCG_Asset<RCG_ItemDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_ItemDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(localized)
    ▼ DropPool / MixDropPools / FilterDropData  ← shown by DropType
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` (direct list) / `MixPool` (combine pools) / `FilterDrop` (tag filter) |
| **Name** | no | Display name (localized). Falls back to first drop's name, then to ID |
| **DropPool** | when DropType=DropPool | Item list + weights (`RCG_CommonDropSetting<RCG_ItemGenData>`) |
| **MixDropPools** | when DropType=MixPool | References other pools with weights |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` (FilterType: Tag / RarityTag / Operator) for dynamic filtering |

## Behavior

### Three Modes
*   **DropPool**: Manually list items + weights.
*   **MixPool**: Combine existing pools by weight. Recursion limited to 10 layers.
*   **FilterDrop**: Conditions (item tag, rarity) with AND / OR / NOT. No conditions = all items uniform.

### Implicit Filtering (runtime)
*   **Locked** (`UnlockData.m_LockedItems.CheckLocked`): skipped.
After stripping, **remaining item weights are renormalized**.

### Preview
Click `ShowDetail` in the editor for the live drop rate list.

## Caveats

*   **DropPool mode** auto-removes invalid item IDs on deserialization (checks `iData.Exist()`).
*   **Inner `DropFilter`** differs from `RCG_CardDropPool`'s `CardDropFilter` — items don't need card-specific filters like SkillTag or EquipmentType.
*   **MixPool cycles** are cut off by `iLayer > 10`.
*   **Locked items** are stripped at runtime; the editor preview may not reflect this runtime logic.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_ItemDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_ItemDropPool>`
*   **Implements**: `UCL.Core.UCLI_ShortName`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Display name | `RCG_LocalizeData` | |
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_ItemGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropDataBase<RCG_ItemGenData, RCG_ItemData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | |

### A.3 Key Methods

*   **`GetDropItems(int)`** — main entry; auto-applies unlock filter then draws N.
*   **`GetDropItemsWithFilterFunc(int, Func)`** — custom filter version.
*   **`GetDropRate(...)`** — normalized weight table.
*   **`DeserializeFromJson`** — cleans invalid IDs.

### A.4 System Interactions

*   **`RCG_ItemData`** — drop target type.
*   **`RCG_ItemGenData`** / **`RCG_ItemDropPoolGenData`** — Asset Entry wrappers.
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedItems`** — unlock filter source.
*   **`FilterDropDataBase<TGenData, TData, TFilter>`** — generic conditional filter base class.

### A.5 Known Issues

*   `// 清理不存在的道具 QWQ` comment marks the cleanup step on deserialize.
*   Recursion cap at 10 layers (`iLayer > 10`).
