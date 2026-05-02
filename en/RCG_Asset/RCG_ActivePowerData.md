---
title: Active Power Data (RCG_ActivePowerData)
description: Character "active abilities" — manually triggered abilities (similar to items, but bound to a character rather than the inventory)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Active Power Data

> Class name: `RCG_ActivePowerData`

## Purpose

**Template for character "active abilities"**. Sits between "equipment" and "items": abilities the player can **manually trigger** in battle (unlike passives), but **bound to a specific character** (unlike items in the shared inventory). Examples: "Healer: once per battle, group heal", "Swordmaster: once per turn, manually trigger crit".

Inherits from `RCG_Asset<RCG_ActivePowerData>`. Implements: `RCGI_Item` / `RCGI_Unloackable`.

## Editor Layout

```
RCG_ActivePowerData: <ID>
    Data (m_Data)               ← main settings (Name / Description / Icon / TargetType / Price / Unlock / ItemUseType)
    ItemEffects                 ← active power effects (OnPlay trigger)
    Preview
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name / Description / Icon** | yes | Name, flavor description, icon |
| **TargetType** | yes | Target selection range when using |
| **Price** | yes | Shop price (if purchasable) |
| **ItemUseType** | yes | `Consume` / `OncePerBattle` / `OncePerTurn` / `Infinite` |
| **UseTimes** | depends | Use count |
| **Unlock** | no | Unlock condition |
| **ItemEffects** | no | Trigger effects (`RCG_CommonEffect` list) |

## Behavior

### Difference from Items
*   **Item**: in shared inventory, any character can use.
*   **Active Power**: bound to a specific character (`RCG_DataService.Ins.m_ActivePowersData`); usually granted when the character joins the party.

### Use / Trigger
`CheckUsable(data)` → all effects' `CheckPlayable` AND.
`TriggerEffect(data)` → fetch and fire `OnPlay` effects (with try-catch).

### Description
Similar to `RCG_ItemData`: auto-composed from effects + UseType marker + UseTimes display.

## Caveats

*   **`ShowInfo()` is empty**: was supposed to use `RCG_ItemInfoPanel`, but commented out (`// QWQ`); currently no-op. Actual UI handled elsewhere.
*   **No ItemType field** (unlike `RCG_ItemData`): already classified as "active power" without further sub-classification.
*   **`InitActivePowers` legacy**: characters used to auto-apply InitActivePowers on join; superseded by UnitSkill system (large commented-out block in code).
*   **No Auto Price tool**: unlike ItemData / EquipmentData, the editor page doesn't provide this.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ActivePowerData.cs`
*   **Inherits**: `RCG_Asset<RCG_ActivePowerData>`
*   **Implements**: `RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**: `EditCharacter`

### A.2 Field Mapping (outer)

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Data` | Data | `ActivePowerData` (nested) | `[SerializeField] protected` |
| `m_ItemEffects` | Effects | `List<RCG_CommonEffect>` | |

`ActivePowerData` nested: `m_Name` / `m_Description` / `m_TargetType` / `m_Price` / `m_Icon` / `m_ItemUseType` / `m_UseTimes` / `m_Unlock`.

### A.3 Key Methods

*   **`AddItem()`** — `RCG_ActivePower.Create(ID)` → `m_ActivePowersData.AddActivePower`.
*   **`CheckUsable(TriggerEffectData)`** — AND on all effects.
*   **`TriggerEffect(TriggerEffectData)`** — fires `OnPlay` effects.
*   **`Description` / `FullDescription` / `GetDescription`** — description generation (`RCG_ActivePower` optional param brings runtime remaining count).
*   **`UseTime`** — per `ItemUseType` returns count (Infinite → 0).

### A.4 System Interactions

*   **`RCG_ActivePower`** — runtime active power instance.
*   **`RCG_DataService.Ins.m_ActivePowersData`** — character-bound storage.
*   **`RCG_CommonEffect`** — trigger effect unit.

### A.5 Known Issues

*   `ShowInfo()` is commented out (`// QWQ`); UI entry point unclear.
*   Legacy `InitActivePowers` auto-apply logic deprecated (replaced by UnitSkill).
