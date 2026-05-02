---
title: Game Setting Data (RCG_GameSettingData)
description: Per-build (Demo / Release) game-level settings — main menu, max unlock level, tutorial reset, shop refresh price
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Game Setting Data

> Class name: `RCG_GameSettingData`

## Purpose

**Build-level game settings** — lets different builds (Demo / Release / Showcase) apply different settings. The system uses `Application.version` as the ID to fetch the corresponding Asset; if no exact-match exists, falls back to `Default`.

Inherits from `RCG_Asset<RCG_GameSettingData>`.

## Editor Layout

```
RCG_GameSettingData: <ID = Application.version>
    GameVersion          ← Default / Demo
    MainMenu             ← reference to main menu setting
    MaxUnlockLevel       ← max unlock level cap
    ResetTutorialOnNewGame ← whether to reset tutorial on new game
    RefreshBlessingShopPrice ← blessing shop refresh price base
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **GameVersion** | yes | `Default` (regular) or `Demo` (locks features / unlock levels) |
| **MainMenu** | yes | Reference to main menu setting (`RCG_MainMenuEntry`) |
| **MaxUnlockLevel** | yes | Max unlock level; level stops growing past this (default 1000) |
| **ResetTutorialOnNewGame** | — | Reset tutorial on each new game; for **showcase builds** |
| **RefreshBlessingShopPrice** | yes | Base for blessing shop refresh; total = (remaining buyable count - 4) × this (default 10) |

## Behavior

### Asset Fetch (`Ins`)
`RCG_GameSettingData.Ins` uses `Application.version` as ID to fetch the corresponding Asset. **So Asset IDs should match the build version** (e.g., `0.1.0` / `0.2.0-demo`); if no exact match, `Util.GetData(version, useDefaultIfMissing = true)` falls back to `Default`.

### Demo Restrictions
When `GameVersion = Demo`, game logic auto-restricts certain features (e.g., forces lower `MaxUnlockLevel`, disables certain chapters). The actual restriction logic is scattered across managers; this file just holds a flag.

## Caveats

*   **ID must correspond to Application.version**: when the build version changes, old Assets won't auto-apply (need to add a new same-name Asset or rely on Default).
*   **`m_GameSetting` commented out**: legacy `RCG_GameInitData.m_GameSetting` referenced this; now uses `Application.version` auto-lookup.
*   **`RefreshBlessingShopPrice` formula**: includes negative case (when remaining ≤ 4, price is 0 or negative); actual price logic must clamp to ≥ 0.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_GameSettingData.cs`
*   **Inherits**: `RCG_Asset<RCG_GameSettingData>`
*   **AssetGroup**: `EditGameSetting`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_GameVersion` | GameVersion | `GameVersion` enum | `Default` / `Demo` |
| `m_MainMenu` | MainMenu | `RCG_MainMenuEntry` | |
| `m_MaxUnlockLevel` | MaxUnlockLevel | `int` | Default 1000 |
| `m_ResetTutorialOnNewGame` | ResetTutorialOnNewGame | `bool` | Default false |
| `m_RefreshBlessingShopPrice` | RefreshBlessingShopPrice | `int` | Default 10 |

### A.3 Key Methods

*   **`Ins` (static)** — `Util.GetData(Application.version, useDefaultIfMissing = true)`.

### A.4 System Interactions

*   **`RCG_MainMenuData`** — main menu setting referenced by `m_MainMenu`.
*   **`RCG_GameInitData`** — game init data; legacy `m_GameSetting` field commented out, now auto-related via `Application.version`.
*   **`RCG_GameSettingGenData`** — Asset Entry; default ID = `"Default"`.

### A.5 Known Issues

*   `Debug.Log("Application.version: ...")` in `Ins` is marked "log!!" — dev-only, must remove before release.
