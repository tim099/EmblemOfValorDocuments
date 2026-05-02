---
title: Common VFX (RCG_CommonVFXData)
description: Generic VFX settings — VFX resource, attached SE, play duration, attach position (DisplayPos)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Common VFX

> Class name: `RCG_CommonVFXData`

## Purpose

**Template for various generic VFX on the battlefield**. Examples: "status layer increase", "charge aura", "overload", "selection ring", "death" — visual effects not falling under attack VFX. Each `RCG_CommonVFXData` contains the VFX resource, attached SE, play duration, and the attach position on the unit.

Inherits from `RCG_Asset<RCG_CommonVFXData>`.

## Editor Layout

```
RCG_CommonVFXData: <ID>
    VFX             ← VFX resource (RCG_VFXResData)
    SE              ← attached SE (optional)
    VFXTime         ← play duration (seconds)
    DisplayPos      ← attach position on unit (EDisplayPos enum)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **VFX** | yes | VFX resource |
| **SE** | no | Attached SE (auto-played on VFX creation) |
| **VFXTime** | yes | Play duration (seconds), default 0.4f |
| **DisplayPos** | yes | Attach position (`EDisplayPos.DisplayPos` is default) |

## Behavior

### `CreateVFX(token)`
1. `m_VFX.IsEmpty` → return null.
2. Create via `m_VFX.CreateVFX()`.
3. Set `vfx.CommonVFXData = this` (for runtime lookback).
4. `m_SE.GetData().PlaySE()` plays the SE.

### `PlayVFX(data, token)`
Creates VFX → sets position (`SetPosition(iData.User)`) → waits `m_VFXTime` seconds.

### Default ID Constants
`RCG_CommonVFXGenData` provides several built-in IDs:
*   `VFX_StatusLayer` — status layer increase
*   `VFX_ChargeEffect` / `VFX_OverloadEffect` — charge / overload
*   `VFX_SelectionRing` — selection ring
*   `CommonVFX_Dead` — death

Also provides static instances `s_ChargeEffect / s_OverloadEffect / s_SelectionRing / s_Dead` to avoid hardcoding ID strings.

## Caveats

*   **`m_SE` auto-plays in `CreateVFX`**: don't call `PlaySE()` again externally to avoid double-play.
*   **`m_VFXTime > 0` waits**: setting 0 means "fire and forget" (returns immediately after creation).
*   **`m_DisplayPos` enum**: determines which layer the VFX attaches to on the unit (foreground / background / center, etc.); see program for exact enum values.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CommonVFXData.cs`
*   **Inherits**: `RCG_Asset<RCG_CommonVFXData>`
*   **AssetGroup**: `EditVFX`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_VFX` | VFX | `RCG_VFXResData` | |
| `m_SE` | SE | `RCG_SEGenData` | |
| `m_VFXTime` | VFXTime | `float` | Default 0.4 |
| `m_DisplayPos` | DisplayPos | `EDisplayPos` enum | |

### A.3 Key Methods

*   **`CreateVFX(token)`** — create + set CommonVFXData + play SE.
*   **`PlayVFX(data, token)`** — create + set position + wait VFXTime.

### A.4 System Interactions

*   **`RCG_VFXResData`** — VFX resource.
*   **`RCG_SEGenData / RCG_SEData`** — SE system.
*   **`RCG_VFX`** — runtime VFX instance.
*   **`RCG_CommonVFXGenData`** — Asset Entry; with built-in IDs and static instances.
*   **`EDisplayPos` (enum)** — display position definition.

### A.5 Known Issues

*   Legacy `DeserializeFromJson` LogError on `LoadType=Resource` is commented out.
