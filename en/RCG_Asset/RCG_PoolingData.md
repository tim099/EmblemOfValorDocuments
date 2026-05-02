---
title: Pooling Data (RCG_PoolingData)
description: Settings for preloading and pooling GameObjects / VFX — type, count, preserve template
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Pooling Data

> Class name: `RCG_PoolingData`

## Purpose

**Object pooling settings**. Specify which prefabs / VFX to pool, how many to prewarm, whether to preserve the template. Reduces runtime object-creation cost (e.g., frequent damage numbers, VFX particles in battle).

Inherits from `RCG_Asset<RCG_PoolingData>`.

## Editor Layout

```
RCG_PoolingData: <ID>
    ResourceType  ▾ PrefabRes / VFXRes
    Prefab    (ResourceType=PrefabRes)
    VFX       (ResourceType=VFXRes)
    PrewarmCount    ← number of pre-built instances
    PreserveTemplate ← whether to preserve the template (true is safer)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **ResourceType** | yes | `PrefabRes` (regular prefab) / `VFXRes` (VFX resource) |
| **Prefab** | when PrefabRes | Corresponding prefab data |
| **VFX** | when VFXRes | Corresponding VFX data (default Addressable path `AttackVFXs/VFX_ArcaneMissileEffect`) |
| **PrewarmCount** | yes | Number of pre-built instances on entry |
| **PreserveTemplate** | — | Whether to preserve the template. `true` = template not lent out (avoids errors when the template gets destroyed or modified) |

## Behavior

### `LoadAsync<T>` / `CreateAsync<T>`
Branches by `ResourceType` to the corresponding `m_Prefab` or `m_VFX` method.

### Pooling Service Integration
Through `RCG_PoolingGenData` (external reference wrapper) → `RCG_ObjectPoolService.Ins`:
*   `GetObjectTemplate<T>` — fetches the template (read-only, not lent).
*   `CreateObject(parent)` — borrows an instance from the pool.
*   `Delete(obj)` — returns to the pool.

### Default IDs
`RCG_PoolingGenData.DefaultID = "ItemDisplay"` (item display); `ItemDisplayerSmallID = "ItemDisplayerSmall"`.

## Caveats

*   **`PreserveTemplate = false`** means the template itself can be lent out — if the template gets modified or destroyed, future fetches break. **Default true is safer.**
*   **`PrewarmCount` doesn't include the template**: actual initial object count = PrewarmCount + 1 (the template).
*   **VFXRes default points to `VFX_ArcaneMissileEffect`**: when creating new pools, remember to swap to your own VFX.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_PoolingData.cs`
*   **Inherits**: `RCG_Asset<RCG_PoolingData>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_ResourceType` | ResourceType | `ResourceType` enum | `PrefabRes` / `VFXRes` |
| `m_Prefab` | Prefab | `RCG_PrefabResData` | `Conditional(PrefabRes)` |
| `m_VFX` | VFX | `RCG_VFXResData` | `Conditional(VFXRes)`; default `VFX_ArcaneMissileEffect` |
| `m_PrewarmCount` | PrewarmCount | `int` | Default 1 |
| `m_PreserveTemplate` | PreserveTemplate | `bool` | Default true |

### A.3 Key Methods

*   **`LoadAsync<T>(token)`** — load resource.
*   **`CreateAsync<T>(token, parent)`** — create instance.
*   **`RCG_PoolingGenData.GetObjectTemplate<T>` / `CreateObject<T> / CreateObject / Delete`** — actual external entry points.

### A.4 System Interactions

*   **`RCG_ObjectPoolService`** — runtime pool management service.
*   **`RCG_PrefabResData / RCG_VFXResData`** — resource loading wrappers.
*   **`PathConst.AttackVFXs`** — VFX default path constant.
