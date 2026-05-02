---
title: Dark Mist Settings (RCG_DarkMistSettingsData)
description: Per-level data for the dark mist mechanic — debuffs to player, buffs to enemies at different mist levels
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Dark Mist Settings

> Class name: `RCG_DarkMistSettingsData`

## Purpose

**Per-level data for the dark mist mechanic**. Dark mist is an environmental pressure that grows as the game progresses; higher mist level = weaker player, stronger enemies. This data defines "what each side gets at each mist level" — usually carried by two shared `CustomStatus` (`DarkMistBuff` / `DarkMistDebuff`), auto-applied at battle start.

Inherits from `RCG_Asset<RCG_DarkMistSettingsData>`.

## Editor Layout

```
RCG_DarkMistSettingsData: <ID>
    DarkMistBuffGenData      ← shared "enemy buff" status ID
    DarkMistDebuffGenData    ← shared "player debuff" status ID
    DarkMistDatas            ← per-level data list (DarkMistLevelData)
        TriggerLevel
        DarkMistBuffStatusData    ← buff effects at this level
        DarkMistDebuffStatusData  ← debuff effects at this level
        Effects                   ← BattleSettings to fire on battle start
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **DarkMistBuffGenData** | yes | Shared "enemy buff status" ID (default `DarkMistBuff`) |
| **DarkMistDebuffGenData** | yes | Shared "player debuff status" ID (default `DarkMistDebuff`) |
| **DarkMistDatas** | yes | Per-level data list (sorted by `TriggerLevel`, **last element wins**) |

Each `DarkMistLevelData` contains:

| Sub-field | Description |
|---|---|
| **TriggerLevel** | This level's threshold (mist level ≥ this triggers this entry) |
| **DarkMistBuffStatusData** | Enemy buff effects at this level (overwritten onto `DarkMistBuff` status) |
| **DarkMistDebuffStatusData** | Player debuff effects at this level (overwritten onto `DarkMistDebuff` status) |
| **Effects** | `RCG_BattleSetting` sequence fired on battle start |

## Behavior

### Get Current Level Data (`GetCurrentDarkMistLevelData`)
Walks the list **from the end backwards**, returns the first entry with `TriggerLevel ≤ iLevel`. So the list **must be sorted ascending by TriggerLevel** (e.g., `[0, 5, 10, 20]`) for correct matching.

### Triggering (`OnTriggerEffect`)
After finding the current level data, queues all `m_Effects` into the action queue.

### Status Override (`UpdateDarkMistEffect`)
Writes the current level's `DarkMistBuffStatusData` to the shared `DarkMistBuff` Status's `m_Effects`. **This is in-place modification of the global Status Asset**, reset before each battle.

## Caveats

*   **DarkMistDatas must be ascending** (by `TriggerLevel`): the lookup logic depends on this; reverse / random order will pick the wrong tier.
*   **Shared Status is dynamically overwritten**: the two `RCG_CustomStatusData` Assets `DarkMistBuff` / `DarkMistDebuff` get their effects overwritten at runtime; don't manually edit them.
*   **Used by multiple sources**: `RCG_GameInitData.m_DarkMistSettings` is a list; can hold multiple sets (e.g., per-difficulty).
*   **Preview is commented out**: editor uses base class default rendering for preview.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_DarkMistSettingsData.cs`
*   **Inherits**: `RCG_Asset<RCG_DarkMistSettingsData>`
*   **AssetGroup**: `EditQuestSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_DarkMistBuffGenData` | DarkMistBuffGenData | `RCG_CustomStatusGenData` | Default `"DarkMistBuff"` |
| `m_DarkMistDebuffGenData` | DarkMistDebuffGenData | `RCG_CustomStatusGenData` | Default `"DarkMistDebuff"` |
| `m_DarkMistDatas` | DarkMistDatas | `List<DarkMistLevelData>` | Per-level data |

### A.3 Key Methods

*   **`OnTriggerEffect(data, level)`** — find current level data → apply effects.
*   **`GetCurrentDarkMistLevelData(level)`** — scan from end for `TriggerLevel ≤ level`.
*   **`UpdateDarkMistEffect(level)`** — overwrite shared Buff Status effects.
*   **`SetDarkMistBuff(status)`** — JSON-serialize / overwrite the entire Buff Asset.

### A.4 System Interactions

*   **`RCG_CustomStatusData`** — the two global statuses `DarkMistBuff` / `DarkMistDebuff`.
*   **`RCG_GameInitData.m_DarkMistSettings`** — list referencing this data.
*   **`RCG_BattleSetting`** — element type of `m_Effects`.

### A.5 Known Issues

*   `Preview` full implementation commented out; uses base default rendering.
*   List must be manually sorted; no auto-sort guarantee — out-of-order data picks the wrong tier.
