---
title: Equipment Drop Pool (RCG_EquipmentDropPool)
description: Defines "which equipment drops, with what weights" — shops, battle rewards, treasure chests draw equipment through this
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Equipment Drop Pool

> Class name: `RCG_EquipmentDropPool`

## Purpose

Defines **which equipment this pool drops and the weight of each**. Battle rewards, shops, treasure chests, boss loot all draw equipment here (weapons, armor, accessories, relics). Same skeleton as card/item pools, with one key difference: **relics cannot drop more than once** — runtime-filtered here.

Inherits from `RCG_Asset<RCG_EquipmentDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_EquipmentDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    Name(localized)
    ▼ DropPool / MixDropPools / FilterDropData
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` / `FilterDrop` |
| **Name** | no | Display name (localized) |
| **DropPool** | when DropType=DropPool | Equipment list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools with weights |
| **FilterDropData** | when DropType=FilterDrop | Inner `DropFilter` (SkillTag / RarityTag / Tag / EquipmentType / Operator) |

## Behavior

### Three Modes
Same as other Drop Pools: DropPool (direct), MixPool (combine), FilterDrop (filter). FilterDrop has an extra **EquipmentType** dimension to filter by weapon/armor/accessory.

### Implicit Filtering (runtime)
*   **Locked** (`UnlockData.m_LockedEquipments.CheckLocked`): skipped.
*   **Relic already owned** (`m_CanDropRepeatedly = false` and player already has same ID equipped): skipped. **This is the mechanism that makes relics appear only once.**
*   **No party member matches the skill** (equipment has `m_SkillTags` but no party member has it): skipped.
After stripping, **remaining equipment weights are renormalized**.

### Preview
`ShowDetail` button shows live drop rates.

## Caveats

*   **Relics' no-repeat dropping** relies entirely on runtime comparison with `RCG_DataService.Ins.m_EquipmentsData.m_Equipments`; preview won't reflect this — needs in-game testing.
*   **Equipment with skill restrictions** can severely shrink (or empty) the pool depending on party composition. Account for all possible parties when designing.
*   **DropPool mode** auto-removes invalid equipment IDs on deserialize.
*   **`CheckRequireSkill` always returns true** in this class — actual filtering is in the `GetDropRate` closure.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_EquipmentDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_EquipmentDropPool>`
*   **Implements**: `UCL.Core.UCLI_ShortName`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Display name | `RCG_LocalizeData` | `[SerializeField] protected` |
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_EquipmentGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_FilterDropData` | Filter | `FilterDropDataBase<RCG_EquipmentGenData, RCG_EquipmentData, DropFilter>` | `Conditional(FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | Default `DropPool` |

### A.3 Key Methods

*   **`GetDropEquipments(int, List<RCG_SkillTagGenData>)`** — main entry; auto-fetches party skills if not specified.
*   **`GetDropRate(List<RCG_SkillTagGenData>, ...)`** — applies unlock + relic + skill checks.
*   **`CheckRequireSkill`** (static) — always returns `true` (signature consistency; actual filtering is in `GetDropRate` closure).
*   **`DeserializeFromJson`** — cleans invalid IDs.

### A.4 System Interactions

*   **`RCG_EquipmentData`** — drop target type.
*   **`RCG_EquipmentGenData`** / **`RCG_EquipmentDropPoolGenData`** — Asset Entry wrappers.
*   **`RCG_DataService.Ins.m_EquipmentsData.m_Equipments`** — player's equipment list, used for relic dedup.
*   **`RCG_DataService.Ins.m_UnlockData.m_LockedEquipments`** — unlock filter.
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — party skill query.

### A.5 Known Issues

*   `CheckRequireSkill` has comment "(should not be needed)" — logic moved to `GetDropRate` closure.
*   Recursion cap at 10 layers.
