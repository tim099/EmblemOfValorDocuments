---
title: Equipment Data (RCG_EquipmentData)
description: Template for character-worn equipment (weapons, armor, accessories, relics) — type, rarity, effects, upgrade branches, no-repeat drop
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Equipment Data

> Class name: `RCG_EquipmentData`

## Purpose

**Template for equipment worn by characters**. Weapons, armor, accessories, relics (special bonus items) all are. Each equipment has its own rarity, type, effects (triggered in battle), upgrade branches, and a "can drop repeatedly" flag.

Inherits from `RCG_Asset<RCG_EquipmentData>`. Implements: `RCGI_Item` / `RCGI_Unloackable`.

## Editor Layout

```
RCG_EquipmentData: <ID>
    Name / Description / Rarity / Tags / EquipmentType / Icon / Price
    Effects                    ← effects triggered in battle
    InitCounters               ← initial counter values
    SkillTags                  ← class restriction
    Unlock                     ← unlock condition
    UpgradeBranch              ← list of equipment this can upgrade into
    EnhenceLevel / HideInCodex / CanDropRepeatedly
    Preview
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Equipment name (localized) |
| **Description** | no | Flavor description |
| **Rarity** | yes | Rarity |
| **Tags** | no | General tags (used by DropPool) |
| **EquipmentType** | yes | `Weapon` / `Armor` / `Accessory` / `Relic` (**Relic: doesn't take a slot, must be on the lead character**) |
| **Icon** | yes | Equipment icon |
| **Price** | yes | Shop sell price; default 500, can be reset by `Auto Price` button (10 × rarity value) |
| **Effects** | no | Effects triggered in battle on various triggers |
| **InitCounters** | no | Initial values for counter-type effects |
| **SkillTags** | no | Class restriction (party must have matching class to drop; OR relation: any one matches) |
| **Unlock** | no | Unlock condition |
| **UpgradeBranch** | no | List of equipment this can upgrade into; empty = not upgradable |
| **EnhenceLevel** | — | Enhance level > 0 means "enhanced version", **hidden from codex** |
| **HideInCodex** | — | Codex hidden (test / enhanced versions are auto-hidden) |
| **CanDropRepeatedly** | — | Can drop repeatedly; relics (`Relic`) usually set false |

## Behavior

### Triggering
`OnTriggerEffect(data, triggerOn)` → fetches effects from `m_Effects` for the trigger and fires in order. `TriggerOnUnitState(triggerOn)` is a quick check.

### Codex Hide Check
`HideInCodex` (property) = `m_HideInCodex || m_EnhenceLevel > 0`: enhanced versions are auto-hidden to avoid showing duplicate "+1 / +2" entries in the codex.

### Description
`GetFullDescription` = `Effects description + m_Description (flavor text)`.

### Auto Price
Editor page has an `Auto Price` button: applies `Price = 10 × Rarity.m_Value` to all equipment.

### Sorting
`RCG_EquimpmentComparer` sorts by "EquipmentType (Weapon=10, Armor=20, Accessory=30)" → "price descending".

## Caveats

*   **Relic (`Relic`) rules**: doesn't take a slot, forced on lead character, `CanDropRepeatedly` usually false (DropPool checks player inventory).
*   **`SkillTags` differs from card SkillTags**: equipment uses OR (any one party member matches), while cards' `RequireSkills` use AND.
*   **Enhanced versions (`EnhenceLevel > 0`) auto-hide from codex**: codex only shows the original.
*   **`NullEquipmentID = "NullEquipment"`** is a reserved system ID meaning "no equipment"; don't use for actual equipment.
*   **Auto Price overwrites manual prices.**

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_EquipmentData.cs`
*   **Inherits**: `RCG_Asset<RCG_EquipmentData>`
*   **Implements**: `RCGI_Item` / `RCGI_Unloackable`
*   **AssetGroup**: `EditItems`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Tags` | Tags | `List<RCG_ItemTagGenData>` | |
| `m_EquipmentType` | EquipmentType | `EquipmentType` enum | Default `Weapon` |
| `m_Icon` | Icon | `RCG_SpriteData` | |
| `m_Price` | Price | `int` | Default 500 |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_InitCounters` | InitCounters | `List<int>` | |
| `m_SkillTags` | SkillTags | `List<RCG_SkillTagGenData>` | |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |
| `m_UpgradeBranch` | UpgradeBranch | `List<RCG_EquipmentGenData>` | |
| `m_EnhenceLevel` | EnhenceLevel | `int` | Default 0 |
| `m_HideInCodex` | HideInCodex | `bool` | |
| `m_CanDropRepeatedly` | CanDropRepeatedly | `bool` | Default `true` |

### A.3 Key Methods

*   **`AddItem()`** — player acquisition: `m_EquipmentsData.AddEquipment(new RCG_Equipment(ID))`.
*   **`OnTriggerEffect(data, triggerOn)`** — battle trigger.
*   **`HideInCodex` (property)** — `m_HideInCodex || m_EnhenceLevel > 0`.
*   **`GetDescription` / `GetFullDescription`** — description generation.
*   **`CreateSelectAssetPage`** — `RCG_EquipmentDataEditorPage.Create()`.

### A.4 System Interactions

*   **`RCG_Equipment`** — runtime equipment instance.
*   **`RCG_DataService.Ins.m_EquipmentsData`** — player equipment storage.
*   **`RCG_EquipmentDropPool`** — drop pool (with `m_CanDropRepeatedly` check).
*   **`RCG_EquipmentInfoPanel`** — detail UI.
*   **`RCG_EquimpmentComparer`** — sort helper.

### A.5 Known Issues

*   `m_CanDropRepeatedly` TODO comment ("需要記錄掉落&排除" / "needs drop tracking & exclusion") hints at history of "no-repeat drop" mechanism changes; currently runtime-checked against player inventory in DropPool.
*   `DeserializeFromJson` auto-pricing line `m_Price = m_Rarity.GetData().m_Value.GetValue(null) * 15` commented out, replaced by Auto Price tool.
