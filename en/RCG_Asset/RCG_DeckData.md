---
title: Deck Data (RCG_DeckData)
description: Definition of a complete deck (cards and configuration) — used for character starting decks, join decks, player deck base
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Deck Data

> Class name: `RCG_DeckData`

## Purpose

**Definition of a complete deck**. Each card + count = one deck. Use cases:
*   `RCG_CharacterData.m_Deck` (starting deck)
*   Character `m_JoinDeck` (deck brought when joining mid-game)
*   `RCG_BattlePresetData.m_Deck` (deck for test battles)
*   Decks granted by special events

Inherits from `RCG_Asset<RCG_DeckData>`. Implements: `RCGI_Unloackable` (unlockable).

## Editor Layout

```
RCG_DeckData: <ID>
    Name           ← deck display name (localized)
    Deck           ← card list (SpawnDeckData, with each card + count)
    Unlock         ← unlock condition
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **Name** | no | Display name (localized); falls back to ID when empty |
| **Deck** | yes | Card list and counts (`SpawnDeckData`) |
| **Unlock** | no | Unlock condition |

## Behavior

### `SelectCard(setting)`
Filters the deck **in order** (not random) by `SelectCardSetting`, returning the first matching card; clones into `RCG_CardBattleData` and returns.

### `SelectCards(count, setting)`
TODO: not yet implemented (`// ToDo` returns null directly).

### `GetAllCards()`
Returns the deck's full `RCG_CardGenData` list.

### Tooltip Infos
`Infos = m_Deck.Infos`: aggregates all status effect descriptions on cards (e.g., "Bleed" / "Burn" status help).

## Caveats

*   **`SelectCards` is unimplemented**: for batch random card draws, this entry is unusable; implement separately or use other utilities.
*   **Default IDs `Default` / `BackUp`**: `RCG_DeckGenData.DefaultID = "Default"`, `BackUpID = "BackUp"` — common naming conventions for "starting deck" and "backup deck".

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_DeckData.cs`
*   **Inherits**: `RCG_Asset<RCG_DeckData>`
*   **Implements**: `RCGI_Unloackable`
*   **AssetGroup**: `EditItems`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_Name` | Name | `RCG_LocalizeData` | |
| `m_Deck` | Deck | `SpawnDeckData` | card list + config |
| `m_Unlock` | Unlock | `RCG_UnlockEntry` | |

### A.3 Key Methods

*   **`SelectCard(setting)`** — picks the first card in order matching the setting; clones to `RCG_CardBattleData`.
*   **`SelectCards(count, setting)`** — **unimplemented** (returns null + TODO).
*   **`GetAllCards()`** — full deck list.
*   **`AllCardsName / Infos / LocalizedName`** — display-related properties.

### A.4 System Interactions

*   **`SpawnDeckData`** — actual deck container.
*   **`RCG_CardBattleData`** — battle card instance.
*   **`SelectCardSetting`** — card selection rules.
*   **`RCG_DeckGenData`** — Asset Entry; `Default` / `BackUp` two constant IDs.

### A.5 Known Issues

*   `SelectCards` not implemented (`// ToDo`).
