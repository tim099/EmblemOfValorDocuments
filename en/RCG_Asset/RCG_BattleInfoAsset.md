---
title: Battle Info Asset (RCG_BattleInfoAsset)
description: Configures the "live stats info" shown on the left of the battle log (cards played this turn, etc.)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Battle Info Asset

> Class name: `RCG_BattleInfoAsset`

## Purpose

**Configures the live stats info shown on the left side of the battle log**. Examples: "Cards played this turn: 3", "Total damage: 120", "Remaining draw slots: 2". Each stat item is an `RCG_BattleInfoAsset` instance; the battle UI sorts them by `m_Order` and displays them in order.

Inherits from `RCG_Asset<RCG_BattleInfoAsset>`.

## Editor Layout

```
RCG_BattleInfoAsset: <ID>
    Enable
    InfoType   ▾ IntVariable
    Variable   ← shown when InfoType=IntVariable
    Order
    HideIfZero
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Enable** | yes | Whether this info is enabled (false → hidden from UI) |
| **InfoType** | yes | Info type; currently only `IntVariable` |
| **Variable** | when InfoType=IntVariable | The integer variable to display (`IntVariable`); can bind to a battle counter |
| **Order** | yes | Display order; smaller comes first |
| **HideIfZero** | — | Hide when value is 0 (avoid clutter from many zero entries) |

## Behavior

### `ShowInfo(data)`
*   `m_HideIfZero = true` and `Variable.GetValue() == 0` → not shown.
*   Otherwise → shown.

### `GetInfo(data)`
Per `InfoType`:
*   `IntVariable` → `m_Variable.GetDescription(iData)` (formatted, with prefix tag).

## Caveats

*   **`InfoType` currently only supports `IntVariable`**: future may add String / Float types but the enum currently has one value.
*   **`m_Order` collisions**: identical Order values don't guarantee display order; manually distinguish values.
*   **HideIfZero ineffective for non-IntVariable**: logic only covers IntVariable case.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_BattleInfoAsset.cs`
*   **Inherits**: `RCG_Asset<RCG_BattleInfoAsset>`
*   **AssetGroup**: `EditBattleSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Enable` | Enable | `bool` | Default `true` |
| `m_InfoType` | InfoType | `InfoType` enum | Currently only `IntVariable` |
| `m_Variable` | Variable | `IntVariable` | `Conditional(IntVariable)` |
| `m_Order` | Order | `int` | Default 0; smaller comes first |
| `m_HideIfZero` | HideIfZero | `bool` | Default `false` |

### A.3 Key Methods

*   **`ShowInfo(TriggerEffectData)`** — whether to show; per `m_HideIfZero` and Variable value.
*   **`GetInfo(TriggerEffectData)`** — formatted string.

### A.4 System Interactions

*   **`IntVariable`** — variable binding source.
*   **Battle log left UI** — consumer of these Assets.

### A.5 Known Issues

*   `InfoType` enum has one value, hinting at planned expansion not yet implemented.
