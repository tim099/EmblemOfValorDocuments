---
title: Stats Asset (RCG_StatsAsset)
description: Bridge between in-game stats and Steam stats — kill count, total damage, etc., for accumulating values
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Stats Asset

> Class name: `RCG_StatsAsset`

## Purpose

**In-game stat definition**. For "accumulating historical values" like "total kills", "total cards played", "total clears" — defined via `RCG_StatsAsset`; can optionally bridge to Steam Stats for auto-reporting.

Inherits from `RCG_Asset<RCG_StatsAsset>`.

## Editor Layout

```
RCG_StatsAsset: <ID>
    HasSteamStat         ← bridge to Steam stat
    SteamUserStat        (when HasSteamStat=true) ← corresponding Steam stat ID
    GameStatsType        ← corresponding in-game stats enum value (None / KillCount / ...)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **HasSteamStat** | — | Whether to bridge to a Steam stat |
| **SteamUserStat** | when HasSteamStat=true | Corresponding Steam stat entry |
| **GameStatsType** | yes | In-game stat type (`GameStatsType` enum) |

## Behavior

### `Create BuiltIn Stats` Editor Tool
The editor page has a button: for each non-`None` value in `GameStatsType` enum, auto-creates the missing Asset (using the enum string as ID) and sets `m_GameStatsType`. **Convenient for syncing program-side enum to Asset-side**.

## Caveats

*   **`GameStatsType` is program-controlled**: adding a new stat type requires changing the enum first, then using the editor tool to create the Asset.
*   **Default ID `None`** (`RCG_StatsEntry.DefaultID`): means "no stat"; don't use as actual stat ID.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_StatsAsset.cs`
*   **Inherits**: `RCG_Asset<RCG_StatsAsset>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_HasSteamStat` | HasSteamStat | `bool` | |
| `m_SteamUserStat` | SteamUserStat | `UCL_SteamUserStatAssetEntry` | `Conditional(HasSteamStat)` |
| `m_GameStatsType` | GameStatsType | `GameStatsType` enum | |

### A.3 Key Methods / Tools

*   **`CreateSelectAssetPage`** — `RCG_StatsAssetEditorPage.Create()`, editor page.
*   **`Create BuiltIn Stats`** (Editor only) — auto-fills missing stats Assets.

### A.4 System Interactions

*   **`UCL_SteamUserStatAssetEntry`** — Steam SDK bridge.
*   **`GameStatsType` (enum)** — program-side stat type enum.
*   **`RCG_StatsEntry`** — Asset Entry; default `None`.
