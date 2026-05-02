---
title: Game Init Data (RCG_GameInitData)
description: Global singleton settings — starting items/equipment/skills, card system settings, pooling, editor settings, dark mist
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Game Init Data

> Class name: `RCG_GameInitData`

## Purpose

**The game's global singleton settings Asset** (single instance, ID fixed to `Default`). Contains:
*   Starting items / equipment / active powers / resources
*   Card system settings (`RCG_CardGameSetting`)
*   Pooling settings (object pools)
*   Prefab resource settings
*   Editor settings
*   Dark mist setting list
*   Equipment type → icon mapping

Inherits from `RCG_Asset<RCG_GameInitData>`.

## Editor Layout

```
RCG_GameInitData: Default
    GameVersion / DefaultLanguage
    CardGameSetting       ← card system global settings
    PrefabResSetting      ← common prefab paths
    PoolingGenDataSetting ← object pool settings
    DarkMistSettings      ← dark mist setting list
    EditorSetting
    InitItems / InitActivePowers / InitEquipments / InitResources
    EquipmentTypeIcon     ← equipment type → icon mapping
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **GameVersion** | yes | Game version (`RCG_GameVersionGenData`) |
| **DefaultLanguage** | yes | Default language |
| **CardGameSetting** | yes | Card system settings (hand size, draw rules, etc.) |
| **PrefabResSetting** | yes | Common prefab path settings |
| **PoolingGenDataSetting** | yes | Pool settings (which prefabs to pool / initial size) |
| **DarkMistSettings** | no | Dark mist setting list (multiple `RCG_DarkMistSettingsData` references) |
| **EditorSetting** | no | Editor-related settings |
| **InitItems / InitActivePowers / InitEquipments / InitResources** | no | Starting items / active powers / equipment / resources for new game |
| **EquipmentTypeIcon** | no | `EquipmentType → RCG_SpriteData` mapping for UI |

## Behavior

### `GameInit()`
Called when starting a new game; adds `m_InitItems` / `m_InitEquipments` / `m_InitActivePowers` to the player (each Asset has its own `AddItem` logic).

### `GetEquipmentIcon(EquipmentType)`
Looks up `m_EquipmentTypeIcon` dictionary; if key missing, LogError and falls back to first entry.

### Static Entries
*   `RCG_GameInitData.Ins` — fetches the `Default` Asset.
*   `CreateInstance()` — re-loads from Asset (`useDefaultIfMissing = false`).

## Caveats

*   **ID fixed to `Default`**: this class expects a single Asset. New IDs won't be auto-loaded (unless explicitly via `Util.GetData`).
*   **`m_GameSetting` commented out**: was once planned to reference GameSettingData here; now uses `Application.version` auto-lookup.
*   **`m_UnlockSetting` commented out**: unlock settings now handled by `RCG_UnlockData`.
*   **`m_InitEquipments` TODO**: "to be moved to character data, characters bring their own initial equipment" — may migrate to `RCG_CharacterData`.
*   **Equipment icon fallback**: when not found, falls back to the first entry — may not be the desired icon; missing entries cause wrong icons.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameInitDatas/RCG_GameInitData.cs`
*   **Inherits**: `RCG_Asset<RCG_GameInitData>`
*   **AssetGroup**: `EditGameSetting`
*   **Constants**: `DefaultID = "Default"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `RCG_GameVersionGenData` | |
| `m_DefaultLanguage` | DefaultLanguage | `UCL_LanguageCodeEntry` | |
| `m_CardGameSetting` | CardGameSetting | `RCG_CardGameSetting` | |
| `m_PrefabResSetting` | PrefabResSetting | `RCG_PrefabResSetting` | |
| `m_PoolingGenDataSetting` | PoolingGenDataSetting | `RCG_PoolingGenDataSetting` | |
| `m_DarkMistSettings` | DarkMistSettings | `List<RCG_DarkMistSettingsGenData>` | |
| `m_EditorSetting` | EditorSetting | `RCG_EditorSetting` | |
| `m_InitItems` | InitItems | `List<RCG_ItemGenData>` | |
| `m_InitActivePowers` | InitActivePowers | `List<RCG_ActivePowerGenData>` | |
| `m_InitEquipments` | InitEquipments | `List<RCG_EquipmentGenData>` | |
| `m_InitResources` | InitResources | `List<RCG_ResourceGenData>` | |
| `m_EquipmentTypeIcon` | EquipmentTypeIcon | `Dictionary<EquipmentType, RCG_SpriteData>` | |

### A.3 Key Methods

*   **`Ins` / `CreateInstance` (static)** — two fetch entries (cached / forced reload).
*   **`GameInit()`** — adds starting items / equipment / abilities on new game.
*   **`GetEquipmentIcon(type)`** — icon lookup; falls back to first on miss.
*   **`CardGameSetting / PoolingGenDataSetting / PrefabResSetting` (static properties)** — quick accessors.

### A.4 System Interactions

*   **`RCG_DataService`** — runtime player data; init items added here.
*   **`RCG_DarkMistSettingsData`** — dark mist setting list elements.
*   **`RCG_CardGameSetting`** — card system settings.
*   **`UCL_LanguageCodeEntry`** — language settings.

### A.5 Known Issues

*   `m_GameSetting` / `m_UnlockSetting` / `m_CardClassBackgroundImages` etc. multiple commented-out blocks, marking design changes.
*   `m_InitEquipments` TODO to move to character data.
