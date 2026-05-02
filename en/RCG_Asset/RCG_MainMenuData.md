---
title: Main Menu Data (RCG_MainMenuData)
description: Container for main menu screen settings (background, buttons, layout); content mostly handled by RCG_MainMenuSetting
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Main Menu Data

> Class name: `RCG_MainMenuData`

## Purpose

**Main menu screen settings** — wraps an `RCG_MainMenuSetting`, allowing different builds / versions to use different main menus (e.g., simplified menu for showcase, locked buttons for Demo).

Inherits from `RCG_Asset<RCG_MainMenuData>`.

## Editor Layout

```
RCG_MainMenuData: <ID = Default>
    MainMenuSetting   ← actual main menu setting (background, buttons, layout, transitions, etc.)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **MainMenuSetting** | yes | The main menu setting body (`RCG_MainMenuSetting`), containing background, buttons, layout details |

## Behavior

This class is just a thin wrapper; actual logic is in `RCG_MainMenuSetting` (not a RCG_Asset subclass; pure data container).

### Static Entries
*   `RCG_MainMenuData.CreateInstance()` — fetches `Default` instance (forced reload).
*   `RCG_MainMenuData.Ins` is commented out; reference flow goes through `RCG_GameSettingData.m_MainMenu`.

### Reference Relations
`RCG_GameSettingData.m_MainMenu` (type `RCG_MainMenuEntry`) references this data; different game versions can specify different MainMenuData via GameSettingData.

## Caveats

*   **Default ID is `Default`**: this class expects a single Asset; for version variants, create multiple IDs.
*   **`m_MainMenuSetting` is `[AlwaysExpendOnGUI]`**: auto-expanded in Inspector for ease of editing.
*   **`Ins` static is commented out**: data fetch goes through GameSettingData; don't call `RCG_MainMenuData.Util.GetData(...)` directly.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_MainMenuData.cs`
*   **Inherits**: `RCG_Asset<RCG_MainMenuData>`
*   **AssetGroup**: `EditGameSetting`
*   **Constants**: `DefaultID = "Default"`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_MainMenuSetting` | MainMenuSetting | `RCG_MainMenuSetting` | `[AlwaysExpendOnGUI]` |

### A.3 Key Methods

*   **`CreateInstance()` (static)** — `Util.GetData(DefaultID, false)`.
*   Constructor defaults `ID = DefaultID`.

### A.4 System Interactions

*   **`RCG_MainMenuSetting`** — actual content.
*   **`RCG_MainMenuEntry`** — Asset Entry wrapper; `RCG_GameSettingData.m_MainMenu` uses this type.

### A.5 Known Issues

*   `Ins` static is commented out, marking the design shift to "fetch via GameSettingData".
