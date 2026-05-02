---
title: Field Effect Drop Pool (RCG_FieldEffectDropPool)
description: Defines "which field effects drop" — source for random battlefield modifiers at map nodes / battle start
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Field Effect Drop Pool

> Class name: `RCG_FieldEffectDropPool`

## Purpose

Defines **which field effects can drop in this context**. Field effects are modifiers **applied to the entire battlefield** (e.g., all units take -1 HP per turn, fire field, darkness shroud). This pool determines candidates and weights when picking a random field effect.

Inherits from `RCG_Asset<RCG_FieldEffectDropPool>`. Implements: `UCL.Core.UCLI_ShortName`.

## Editor Layout

```
RCG_FieldEffectDropPool: <ID>
    DropType  ▾ DropPool / MixPool      ← no FilterDrop
    ▼ DropPool / MixDropPools
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DropType** | yes | `DropPool` / `MixPool` (**only two modes**) |
| **DropPool** | when DropType=DropPool | Field effect list + weights |
| **MixDropPools** | when DropType=MixPool | References other pools |

## Behavior

Same structure as `RCG_StatusDropPool` — only DropPool / MixPool modes.

## Caveats

*   **No FilterDrop mode**.
*   **Enum is `DropType`** (not `EDropType`).

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_DropSettings/RCG_FieldEffectDropPool.cs`
*   **Inherits**: `RCG_Asset<RCG_FieldEffectDropPool>`
*   **AssetGroup**: `EditDropSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DropPool` | Drop pool | `RCG_CommonDropSetting<RCG_FieldEffectGenData>` | `Conditional(DropPool)` |
| `m_MixDropPools` | Mix pools | `List<MixDropPoolData>` | `Conditional(MixPool)` |
| `m_DropType` | DropType | `DropType` enum | Only `DropPool` / `MixPool` |

### A.3 System Interactions

*   **`RCG_FieldEffectData`** — drop target.
*   **`RCG_FieldEffectGenData`** / **`RCG_FieldEffectDropPoolGenData`** — Asset Entry wrappers.
