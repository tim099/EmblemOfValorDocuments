---
title: Card create (selected)
description: Generate cards from a candidate list with player choice (or random) and a destination
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card create (selected)

> Class: `RCG_CreateSelectedCardSetting`

## Purpose
Generate cards from a **candidate pool** with a player-choice step (or random pick). Adds a "**pick from many**" stage compared to plain **Card create**. Examples:
*   "Choose 1 of 3 random rewards to add to hand"
*   "Pick any card from the entire deck to hand"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CreateType** | Yes | Source:<br>• **Default** — pick from the `CardGenDatas` list<br>• **FromDeck** — pick from a specified deck |
| **CreateCardType** | Yes | Destination (same as Card create): hand / deck / discard / deck-top. |
| **CreateCount** | Yes | Max cards the player may pick. |
| **IsRandomSelect** | — | Checked = **random** pick (no UI); unchecked = manual choice. |
| **CardGenDatas** | When CreateType=Default | Candidate card list. |
| **Deck** | When CreateType=FromDeck | Candidate deck data (`RCG_DeckGenData`). |

## Behaviour
*   `Default` + manual → opens chooser UI from `CardGenDatas`.
*   `FromDeck` + manual → opens chooser from the deck's full content.
*   `IsRandomSelect = true` → no UI; randomly picks N.
*   After selection, inserts into `CreateCardType` destination.

## Notes
*   **vs Card create**: Card create makes a fixed card; this version **prompts the player to choose** (unless `IsRandomSelect`).
*   **CardGenDatas duplicates**: deduped in `Infos`; the chooser UI may still show duplicates.
*   **AI usage**: chooser UI is player-only; AI should use `IsRandomSelect = true`.
*   **FromDeck pulls the full deck**: can be a lot of options — consider filtering up-stream.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CreateSelectedCardSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CreateSelectedCardSetting` → "Card create (selected)"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CreateType` | CreateType | enum (file) | — | `Default` / `FromDeck` |
| `m_CreateCardType` | CreateCardType | `CreateCardType` (shared) | — | 4 destinations |
| `m_CreateCount` | CreateCount | `IntVariable` | — | Default 1 |
| `m_IsRandomSelect` | IsRandomSelect | `bool` | — | |
| `m_CardGenDatas` | CardGenDatas | `List<RCG_CardGenData>` | — | `[Conditional("m_CreateType", false, Default)]` |
| `m_Deck` | Deck | `RCG_DeckGenData` | — | `[Conditional("m_CreateType", false, FromDeck)]` |

### A.3 Key methods
*   **`Infos`**: `Default` mode dedupes via HashSet then adds per-card; `FromDeck` mode adds deck info.
*   **`CardDatas` (private)**: returns candidate `RCG_CardData` list per CreateType.
*   **`AddAction`** (beyond shown range): selection + destination insert (branches on IsRandomSelect).

### A.4 Cross-system interactions
*   **`RCG_DeckGenData`**: deck source.
*   **`CreateAction.AddSelectCardAction / CreateCard`**: UI selection + card insert builders.
