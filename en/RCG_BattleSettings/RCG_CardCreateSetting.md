---
title: Card create
description: Generate a specified card and add it to hand / deck / discard pile, multi-count supported
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card create

> Class: `RCG_CardCreateSetting`

## Purpose
**Generate new cards** in a chosen location. Examples:
*   "Add 2 Burn cards to discard"
*   "Add an Enhanced version on top of deck"
*   "Generate a Taunt card to deck"

For **player choice from multiple options**, use **Card create (selected)** (`RCG_CreateSelectedCardSetting`) instead.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **CardData** | Yes | Card template (`RCG_CardGenData`). |
| **IsChanted** | Chant cards only | Chanted state (only shown when CardData is a chant card). |
| **CreateCardType** | Yes | Destination:<br>• **AddToHandCard** — to hand<br>• **AddToDeck** — random deck position (default)<br>• **AddToDiscardPile** — to discard<br>• **AddToDeckTop** — top of deck |
| **CreateCount** | Yes | Number of copies; supports variables. |

## Behaviour
*   Description: "**Generate {count} {cardName}**" (i18n `CardCreateEffectDes_{CreateCardType}`).
*   Per copy → `RCG_CardBattleData.CreateCard(template, IsChanted)` and inserts at the destination.
*   Created cards are recorded in `iData.m_TriggerData.m_CreatedCards` for later reference.

### Card fusion
Two Card create cards fused: **CreateCount adds**. ("Generate 1 X" + "Generate 2 X" → "Generate 3 X")

## Notes
*   **Invalid CardData ID**: `CardData.GetData()` returns null at runtime; name shows empty. Verify ID is registered.
*   **IsChanted only for chant cards**: gated by reflection `IsChantCard()`.
*   **AddToDeckTop has no animation**: players see the card next draw, not on creation. Use AddToHandCard for visible feedback.
*   **vs Card create (selected)**: this version has no player choice; "selected" version shows a chooser.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardCreateSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, implements `UCLI_FieldOnGUI`, `[System.Serializable]`
*   **i18n class key**: `RCG_CardCreateSetting` → "Card create"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_CardData` | CardData | `RCG_CardGenData` | — | |
| `m_RuntimeCardData` | (hidden) | `RCG_CardBattleDataPointer` | — | `[UCL_HideOnGUI]`; for itemization references |
| `m_IsChanted` | IsChanted | `bool` | — | `[Conditional(nameof(IsChantCard))]` |
| `m_CreateCardType` | CreateCardType | `CreateCardType` (enum) | — | 4 destinations |
| `m_CreateCount` | CreateCount | `IntVariable` | — | `[UCL_FieldOnGUI]` |

### A.3 Key methods
*   **`CardData` (property)**: prefers `m_RuntimeCardData`, falls back to `m_CardData.GetData()`.
*   **`OnGUI`**: custom field rendering with `RCG_StaticFunctions.LocalizeFieldName` callback.
*   **`AddAction`**: loops `CreateCount` times → `RCG_CardBattleData.CreateCard(...)` → records to `m_CreatedCards` → queues `CreateAction.CreateCard(card, m_CreateCardType)`.
*   **`Fusion(other)`**: clone + `IntVariable.FuseAdd(m_CreateCount)`; **doesn't check CardData equality** (potential gotcha).
*   **`IsChantCard()` (private)**: reflection target for `Conditional` on `m_IsChanted`.

### A.4 Cross-system interactions
*   **`RCG_CardBattleData.CreateCard(...)`**: instance creation.
*   **`CreateAction.CreateCard`**: actual Action class.
*   **`RCG_CardBattleDataPointer`**: runtime card reference (used for itemization).

### A.5 Known issues
*   `m_RuntimeCardData` serialization is commented out (using base default); verify if persistence is still needed.
*   `Fusion` doesn't check `m_CardData` equality — fusing different cards could merge counts onto the first card's CardData.
