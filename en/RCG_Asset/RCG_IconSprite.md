---
title: TMP Icon Sprite (RCG_IconSprite)
description: Inline icons for TextMeshPro — small icons in UI descriptions (HP / armor / card, etc.)
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# TMP Icon Sprite

> Class name: `RCG_IconSprite`

## Purpose

**Inline icons for TextMeshPro**. Card descriptions, status help, and battle log frequently show "+5 HP" / "-3 armor" with small icons next to them — those icons come from TMP's `<sprite name=...>` tag inserted into text. This Asset is the source of those icons. The system auto-bundles all `RCG_IconSprite` Assets into one `TMP_SpriteAsset` for text rendering.

Inherits from `RCG_Asset<RCG_IconSprite>`.

## Editor Layout

```
RCG_IconSprite: <ID>
    Icon       ← icon sprite
    Scale      ← scale (in TMP display)
    BearingX   ← horizontal baseline offset
    BearingY   ← vertical baseline offset (default 0.85)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Icon** | yes | Icon sprite (`RCG_SpriteData`); default `Status_BloodDrain` |
| **Scale** | yes | Scale in TMP (default 1) |
| **BearingX** | — | Horizontal baseline offset |
| **BearingY** | — | Vertical baseline offset (default 0.85, aligns with TMP text baseline) |

## Behavior

### TMPKey
Each IconSprite auto-generates a TMP tag: `<sprite name={ID}>`. Use this in text for inline icon display.

### Auto-Bundle into TMP_SpriteAsset
`InitSpriteAsset(token)` scans all `RCG_IconSprite` at startup and bundles them into a single `TMP_SpriteAsset`; auto-refreshes when modules reload (`UCL_ModuleService.OnLoadedModule`).

### `Save()` Triggers RefreshSpriteAsset
Saving this Asset in editor auto-triggers `RefreshSpriteAsset()` to rebuild the bundle and see changes immediately.

### Built-in Static References
*   `EffectIcon_Target` / `EffectIcon_CountDown` / `EffectIcon_Armor` / `EffectIcon_Health` / `EffectIcon_Card` / `EffectIcon_Die` / `EffectIcon_Energy` / `EffectIcon_Magnifier`
These are common icons used in code, accessed directly via static properties.

### Editor Tools
*   **`Output sprite sheet`** (Editor only): exports the entire sprite sheet image (composite atlas).
*   **`Refresh SpriteAsset`**: manually re-bundles.

## Caveats

*   **Bundling is async**: `InitSpriteAsset` uses `s_IsRefreshingSpriteAsset` flag for mutex; concurrent calls wait for the previous one to finish.
*   **Default `BearingY = 0.85`**: aligns with TMP font baseline; tweaking can make icons drift up/down.
*   **Default ID `AbsorbShield`** (`RCG_IconSpriteGenData.DefaultID`).
*   **`m_Disable` is commented out**: was once planned to skip disabled icons; currently all icons are bundled.
*   **`LocalizeName` behavior**: UI mode → TMPKey (icon), non-UI mode → i18n translation (plain text); the card description system switches based on `RCG_BattleSetting.IsShowOnUI`.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_IconSprite.cs`
*   **Inherits**: `RCG_Asset<RCG_IconSprite>`
*   **AssetGroup**: `EditTags`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Icon` | Icon | `RCG_SpriteData` | Default `Status_BloodDrain` |
| `m_Scale` | Scale | `float` | Default 1 |
| `m_BearingX` | BearingX | `float` | Default 0 |
| `m_BearingY` | BearingY | `float` | Default 0.85 |

### A.3 Key Methods (including statics)

*   **`InitSpriteAsset(token)` (static async)** — startup bundling; subscribes OnLoadedModule.
*   **`GenerateSpriteAsset(token)` (static)** — loads all icons and creates `TMP_SpriteAsset`.
*   **`RefreshSpriteAsset / RefreshSpriteAssetAsync`** — re-bundle (mutex on repeated calls).
*   **`LoadIconSprites(token)` (private static)** — helper to read all textures + names + icons.
*   **`Save()` (override)** — triggers RefreshSpriteAsset.
*   **`TMPKey` (property)** — `<sprite name={ID}>`.
*   Multiple static `EffectIcon_*` shortcut references.

### A.4 System Interactions

*   **`TMP_SpriteAsset`** — TextMeshPro icon resource.
*   **`RCG_TMPTools`** — bundling and tools (`CreateSpriteAsset / RefreshSpriteAsset / CreateIconSpriteSheetEditor`).
*   **`RCG_SpriteData`** — source sprite.
*   **`UCL_ModuleService.OnLoadedModule`** — module reload auto-refresh.
*   **`RCG_BattleSetting.IsShowOnUI`** — text description mode switch.
*   **`RCG_IconSpriteGenData`** — Asset Entry; default `AbsorbShield`.

### A.5 Known Issues

*   `m_Disable` / `CheckSpriteAsset / GenerateSpriteAsset (sync ver)` etc. multiple commented blocks marking the legacy sync bundling flow → async.
*   `s_IsRefreshingSpriteAsset` is a global static flag; if a task throws an exception without resetting, future refreshes may hang.
