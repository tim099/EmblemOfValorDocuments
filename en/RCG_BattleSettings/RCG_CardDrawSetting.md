---
title: Card draw
description: Draw cards from deck top, or pick from deck / discard pile, with optional tag filters
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card draw

> Class: `RCG_CardDrawSetting`

## Purpose
**Draw cards** to hand. Supports drawing from deck top (normal), picking from deck, or picking from discard pile.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **DrawCardNum** | Yes | Cards to draw (variable). Auto-clamped to `[0, hand space]`. |
| **DrawType** | Yes | Source:<br>• **FromDeckTop** — normal draw (default)<br>• **PickFromDeck** — pick from deck<br>• **PickFromDiscardPile** — pick from discard (recover) |
| **CardTags** | No | Restrict to cards with these tags. |
| **DrawCardNotIncludedTags** | No | Inverse: cards **without** these tags. |

## Behaviour
*   **FromDeckTop**: directly draws N to hand (clamped by hand space).
*   **Pick\*** modes: open a selection UI for the player.
*   Description: "**Draw {count} {tag} cards**" (i18n `DrawCardIcon_{DrawType}`).

### Card fusion
Two Card draw cards fuse by adding `DrawCardNum` (**doesn't check DrawType / CardTags equality** — potential gotcha).

## Notes
*   **Hand space cap**: drawing more than space allows just silently drops the excess (no auto-discard).
*   **Tag filter + empty deck**: with no matching cards in deck, **the system doesn't auto-shuffle discard**, so you may draw fewer than N.
*   **Pick\* modes invalid for AI**: the selection UI is player-only. Use `FromDeckTop` for AI-driven draws.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDrawSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_CardDrawSetting` → "Card draw"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_DrawCardNum` | DrawCardNum | `IntVariable` | — | `[UCL_FieldOnGUI]` |
| `m_DrawType` | DrawType | `DrawType` (file enum) | — | 3 sources |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | — | |
| `m_DrawCardNotIncludedTags` | DrawCardNotIncludedTags | `bool` | — | |

### A.3 Key methods
*   **`AddAction`**: `Mathf.Clamp(m_DrawCardNum, 0, RCG_Player.Ins.CardSpace)` → branch on `m_DrawType`:
    *   `FromDeckTop` → `CreateAction.AddDrawCardAction`.
    *   `PickFromDeck` → `CreateAction.AddSelectCardAction(CardPos.Deck, ...)` + `AddPickFromDeckAction`.
    *   `PickFromDiscardPile` → same with `CardPos.DiscardPile`.
*   **`Fusion`**: clone + `IntVariable.FuseAdd(m_DrawCardNum)`; **doesn't check other fields**.
*   **`LocalizeKey`** → `"DrawCardIcon_" + m_DrawType.ToString()`.

### A.4 Cross-system interactions
*   **`SelectCardSetting`**: shared filter helper (defined in `RCG_CardDiscardSetting.cs`).
*   **`CreateAction.AddSelectCardAction / AddPickFromDeckAction / AddDrawCardAction`**: draw action builders.
*   **`RCG_Player.Ins.CardSpace`**: hand space cap.
