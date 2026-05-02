---
title: Card Drop Pool (RCG_CardDropPool)
description: Defines a set of "which cards drop, with what weights" — battle rewards, shops, enhance branches all draw cards through this
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card Drop Pool

> Class name: `RCG_CardDropPool`

## Purpose

Defines **which cards this pool can drop and the weight of each**. Battle rewards, shop restocks, merchant specials, and event drops all randomly draw from a `RCG_CardDropPool`. The same pool can be referenced by many sources — change it once and the change applies everywhere.

Inherits from `RCG_Asset<RCG_CardDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_CardDropPool: <ID>
    DropType  ▾ DropPool / MixPool / FilterDrop
    ▼ Display Name
        Name(localized)
    ▼ DropPool / MixDropPools / FilterDropData  ← shown based on DropType
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` (list cards + weights) / `MixPool` (combine other pools) / `FilterDrop` (filter by tags) |
| **Name** | no | Display name (localized). When blank, `GetShortName()` falls back to the first drop's name, then to ID |
| **DropPool** | when DropType=DropPool | Card list + weights (`RCG_CommonDropSetting<RCG_CardGenData>`) |
| **MixDropPools** | when DropType=MixPool | References other drop pools with weights — final result is the weighted average |
| **FilterDropData** | when DropType=FilterDrop | Uses `CardDropFilter` conditions (tag, rarity, etc.) to dynamically pick matching cards. Empty conditions = all cards |

## Behavior

### Three Modes
*   **DropPool (direct)**: Manually list each card + weight. Most explicit, good for curated pools.
*   **MixPool (combined)**: Combines multiple existing pools with weights, avoiding duplicate maintenance. Weights stack (outer × inner). Recursion limited to 10 layers to prevent cycles.
*   **FilterDrop (tag-based)**: Picks matching cards from the **entire card pool**; all matches share equal weight. Conditions support nested AND / OR / NOT.

### Implicit Filtering (auto-applied at runtime)
Even with weight 0.5 explicitly set, runtime can still strip cards from the pool:
*   **Locked** (`UnlockData.CheckCardLocked`): skipped.
*   **No party member can use it** (`CheckRequireSkill`): card has skill requirements but no party member has the matching skill — skipped.
*   **Main menu / non-battle**: skill check skipped (avoids spurious UI filtering).
After stripping, **remaining card weights are renormalized** (sum back to 1).

### Preview
Click `ShowDetail` on the right of the editor to see the live final drop rate list under current conditions.

## Caveats

*   **In DropPool mode**, deserialization **automatically removes invalid card IDs** (renamed / deleted). MixPool mode is similar (removes missing sub-pools). A `Remove Invalid DropPool` LogError in the console means this pool used to reference a bad ID.
*   **Empty FilterDrop conditions** = all cards drop with equal weight. Unless that's intentional, always set conditions.
*   **MixPool cycles** are cut off by `iLayer > 10` (returning empty); if a pool is dropping nothing, check the mix chain for cycles.
*   **RCG_CardFilter cache**: FilterDrop queries go through `RCG_CardFilter.GetCards`, which is cached. After adding new cards, if preview is wrong, trigger cache clear via `RCG_CardData.OnLoadModule`.

---

## Appendix: Programmer Reference

> Below uses internal terminology — audience shifts to programmers and AI agents. Trust the upper section first.

### A.1 Class Info

*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_CardDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_CardDropPool>`
*   **Implements**: `UCL.Core.UCLI_ShortName`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Localize Key | Notes |
|---|---|---|---|---|
| `m_Name` | Display name | `RCG_LocalizeData` | `Name` | `[SerializeField] protected` |
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_CardGenData>` | — | `Conditional(m_DropType == DropPool)` |
| `m_MixDropPools` | Mix pool list | `List<MixDropPoolData>` | — | `Conditional(m_DropType == MixPool)` |
| `m_FilterDropData` | Filter conditions | `FilterDropData` (nested) | — | `Conditional(m_DropType == FilterDrop)` |
| `m_DropType` | DropType | `EDropType` enum | `DropType` | Default `DropPool` |

### A.3 Key Methods

*   **`GetDropCards(int, bool)`** → primary external entry, returns `iDropCount` random cards.
*   **`GetDropCardsWithFilterFunc(int, Func)`** → custom filter version (e.g., specifying rarity).
*   **`GetDropRate(bool, CheckDropConditionData, int)`** → returns the final weight table (sums to 1); `iIsFilterSkill = true` auto-applies `CheckRequireSkill`.
*   **`CheckRequireSkill(RCG_CardGenData)`** (static) → three-layer check: runtime → main menu skip → unlock check → party skill match.
*   **`DeserializeFromJson`** → cleans invalid IDs on load (handled per-mode for DropPool / MixPool).
*   **`Preview`** → editor preview UI; expandable to show full drop rate table.

### A.4 System Interactions

*   **`RCG_CommonDropSetting<T>`** — generic drop container; `m_DropPool` uses it directly.
*   **`RCG_CardGenData`** — card ID wrapper; drop results are `List<RCG_CardGenData>`.
*   **`RCG_CardDropPoolGenData`** — Asset Entry wrapper; other Assets reference this pool with this type.
*   **`RCG_CardFilter`** — query entry for FilterDrop mode.
*   **`RCG_DataService.Ins.m_UnlockData`** — runtime lock query.
*   **`RCG_CharacterDataService.Ins.GetAllSkillTags()`** — party skill query.

### A.5 Known Issues

*   `MixPool` doesn't normalize weight sums internally (raw `aWeight * aDrop.m_Weight` accumulation); final normalization happens in `GetDropRate()` (sum=1).
*   Recursion cap of 10 layers (`iLayer > 10` returns empty) is a magic number.
