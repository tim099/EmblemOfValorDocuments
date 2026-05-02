---
title: Resource Type (RCG_ResourceTypeData)
description: In-game resource definitions (Gold / Supply / Spirit / Blessing) — name, description, icon
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Resource Type

> Class name: `RCG_ResourceTypeData`

## Purpose

**Definitions for in-game accumulable resource types**. Gold, Supply (dark mist countermeasure item), Spirit (souls), Blessing — different IDs of `RCG_ResourceTypeData`. This data defines their display name, description, and icon.

Inherits from `RCG_Asset<RCG_ResourceTypeData>`.

## Editor Layout

```
RCG_ResourceTypeData: <ID>
    Name(localized)
    Description(localized)
    IconSprite      ← reference to RCG_IconSprite
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | yes | Resource name (localized) |
| **Description** | no | Resource description (used in tooltip) |
| **IconSprite** | yes | Icon reference (`RCG_IconSpriteGenData`) |

## Behavior

### LocalizeName Switching
*   `RCG_BattleSetting.IsShowOnUI = false` → returns `m_Name.Name` (plain text).
*   `IsShowOnUI = true` → returns `m_IconSprite.TMPKey` (sprite tag for TextMeshPro).

### Built-in Four Resources (program constants)
*   `Gold` (`ResourceType_Gold`)
*   `Supply` (`ResourceType_Supply`) — also relates to "dark mist" usage
*   `Spirit` (`ResourceType_Spirit`)
*   `Blessing` (`ResourceType_Blessing`)

## Caveats

*   **ID naming convention**: `ResourceType_<EnumName>` (e.g., `ResourceType_Gold`).
*   **`enum ResourceType` (in-file)** lists only three (Gold / Supply / Spirit); Blessing uses a string directly without being in the enum.
*   **Relation to `RCG_DifficultyData.m_PriceMult / m_SoulPriceMult`**: price multipliers affect Gold / Spirit shop prices.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_ResourceTypeData.cs`
*   **Inherits**: `RCG_Asset<RCG_ResourceTypeData>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Description` | Description | `RCG_LocalizeData` | |
| `m_IconSprite` | IconSprite | `RCG_IconSpriteGenData` | |

### A.3 Key Properties

*   **`LocalizeName`** — UI mode → TMPKey; else Name.
*   **`Description`** — `m_Description.Name`.
*   **`Icon`** — `m_IconSprite.GetData().IconSprite`.

### A.4 System Interactions

*   **`RCG_ResourceTypeGenData`** — Asset Entry; includes 4 static defaults (Gold / Supply / Spirit / Blessing).
*   **`RCG_IconSpriteGenData`** — icon resource reference.
*   **`RCG_DataService.Ins.GetResource(...)`** — runtime resource accessor.
*   **`RCG_BattleSetting.IsShowOnUI`** — toggles LocalizeName mode.
