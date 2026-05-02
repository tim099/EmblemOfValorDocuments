---
title: Attack VFX (RCG_AttackVFXData)
description: VFX settings for card attacks — launch VFX, hit VFX, charge-toward-enemy; with preload mechanism
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Attack VFX

> Class name: `RCG_AttackVFXData`

## Purpose

**Template for card attack VFX**. An attack card can specify:
*   **Launch VFX** (`m_VFX`): plays at the user's position when attacking
*   **Hit VFX** (`m_HitVFX`): plays at the enemy's position on damage settlement
*   **Whether to charge toward the enemy** + animation timing (delay, move time, dwell, retreat)

Inherits from `RCG_Asset<RCG_AttackVFXData>`.

## Editor Layout

```
RCG_AttackVFXData: <ID>
    AttackVFXSetting (m_AttackVFXSetting)
        VFX                       ← launch VFX
        HitVFX                    ← hit VFX
        MoveTowardEnemy           ← whether to charge toward the enemy
        MoveTowardEnemySetting    (when true) ← animation timing
    Note                          ← memo
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **VFX** | yes | Launch VFX; default Addressable `AttackVFXs/VFX_AttackEffect` |
| **HitVFX** | no | Hit VFX |
| **MoveTowardEnemy** | — | Whether to move to the enemy when attacking |
| **MoveTowardEnemySetting** | when MoveTowardEnemy=true | Animation timing: `AttackAnimDelay` / `MoveTime` (0.5) / `WaitTime` (0.8) / `MoveBackTime` (0.25) / `OffsetX` |
| **Note** | no | Memo (for editor use only, not displayed at runtime) |

## Behavior

### Preload (`PreloadData`)
Called when card data loads, runs `AttackVFXSetting.PreloadData(token)`: preloads `m_VFX` and `m_HitVFX` to memory to avoid first-play stutter in battle.

### Charge Toward Enemy
When `m_MoveTowardEnemy = true`:
1. Wait for `m_AttackAnimDelay`.
2. Move to the enemy (`m_MoveTime`).
3. Dwell at the enemy (`m_WaitTime`) — VFX plays, damage settles.
4. Retreat to original position (`m_MoveBackTime`).
5. `m_OffsetX` controls the horizontal offset of the dwell position (mirrored for player vs. enemy — player +X, enemy -X).

## Caveats

*   **Default ID `AttackEffectPlain`** (`RCG_AttackVFXGenData.DefaultID`): the built-in "plain attack" VFX.
*   **`HitVFX` path constant**: `PathConst.HitVFXs`, default empty (many attacks don't need a separate hit VFX).
*   **`Note` field is purely a memo**: no runtime effect, just for designers.
*   **Even after preload, first-time stutter may still occur**: if using Resource path instead of Addressable, preload effectiveness is limited.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_AttackVFXData.cs`
*   **Inherits**: `RCG_Asset<RCG_AttackVFXData>`
*   **AssetGroup**: `EditVFX`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_AttackVFXSetting` | AttackVFXSetting | `AttackVFXSetting` (nested) | main data |
| `m_Note` | Note | `string` | |

`AttackVFXSetting` contains `m_VFX` / `m_HitVFX` / `m_MoveTowardEnemy` / `m_MoveTowardEnemySetting`.
`MoveTowardEnemySetting` contains `m_AttackAnimDelay` / `m_MoveTime` / `m_WaitTime` / `m_MoveBackTime` / `m_OffsetX`.

### A.3 Key Methods

*   **`AttackVFXSetting.PreloadData(token)`** — preloads m_VFX + m_HitVFX.

### A.4 System Interactions

*   **`RCG_VFXResData`** — VFX resource wrapper.
*   **`PathConst.AttackVFXs / HitVFXs`** — Addressable path constants.
*   **`RCG_AttackVFXGenData`** — Asset Entry; default `AttackEffectPlain`.
*   **Battle animation system** — runtime applies `MoveTowardEnemySetting` timing parameters.

### A.5 Known Issues

*   Legacy ID migration logic (`m_ID == "AttackEffect"` → `DefaultID`) is commented out.
