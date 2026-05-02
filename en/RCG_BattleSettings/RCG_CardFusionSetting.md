---
title: Card fusion
description: Player picks hand cards to fuse into a new card; originals are banished
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card fusion

> Class: `RCG_CardFusionSetting`

## Purpose
**Lets the player pick hand cards to fuse**: chosen cards are banished, and a new fused card is added to hand. Examples:
*   Alchemist-style synthesis cards
*   "Fuse two attack cards into an enhanced attack"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectHandCardSetting** | Yes | Embedded selector — controls UI conditions, count, restrictions. |

## Behaviour
*   Triggers the embedded selector first.
*   Then:
    1. Computes the fusion result via `CardFusion()`.
    2. Builds new `RCG_CardBattleData` from the result.
    3. **Banishes** all chosen cards (not discarded).
    4. Adds new card to hand.
*   Description: "**Fuse selected cards**" (i18n `CardFusion_Des`).

## Notes
*   **Fusion logic lives in `CardFusion()`**: card-system level — to know what fuses to what, see `RCG_CardData` / `IList<RCG_CardData>.CardFusion()`.
*   **Originals are banished, not discarded**: discard-related effects don't fire.
*   **No fuseable candidates**: selection UI still opens but with nothing to pick. Set min count in selector.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardFusionSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CardFusionSetting` → "Card fusion"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | |

### A.3 Key methods
*   **`AddAction`**: invokes `m_SelectHandCardSetting.AddAction(InsertInOrder)`, then in subsequent ActionTrigger:
    1. Collect `iData.SelectedHandCards` → `RCG_CardData` list.
    2. `aSelectedCards.CardFusion()` → fused `RCG_CardData`.
    3. `RCG_CardBattleData.CreateCard(fused, false)`.
    4. `CreateAction.AddDiscardCardActions(..., RemoveType.Banish)` for originals.
    5. `CreateAction.CreateCard(newCard, AddToHandCard)`.
*   **`GetDescriptionFormat`** → `CardFusion_Des` wrapping selector format.

### A.4 Cross-system interactions
*   **`IList<RCG_CardData>.CardFusion()`**: extension method with actual fusion logic.
*   **`RCG_SelectHandCardSetting`**: selection UI.
*   **`CreateAction.AddDiscardCardActions / CreateCard`**: banish + create builders.
