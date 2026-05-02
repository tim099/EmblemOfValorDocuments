---
title: Copy
description: Copy selected hand cards into hand / deck / discard pile
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Copy

> Class: `RCG_CopySetting`

## Purpose
**Copy selected hand cards** — produces new copies and inserts them in the chosen location. Examples:
*   "Copy this card to hand"
*   "Copy chosen cards to top of deck"

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectHandCardSetting** | Yes | Card selector (supports `SkipSelect` / `ThisCard` / standard). |
| **CreateCardType** | Yes | Destination: hand / deck / discard / deck-top. |

## Behaviour
*   Triggers selector, then for each selected card → `CopyCard()` → `CreateAction.CreateCard(copy, CreateCardType, InsertInOrder)`.
*   Description varies by `SelectType`:
    *   `SkipSelect` → "Copy"
    *   `ThisCard` → "Copy this card"
    *   Other → "{selector desc} and copy"
*   If `CreateCardType ≠ AddToHandCard`, appends location info.

## Notes
*   **Copies don't carry chant state**: a copy is a fresh card; original's chant progress doesn't transfer (unless explicitly handled).
*   **Copying enhanced cards**: `CopyCard()` includes enhancements; the copy is also enhanced (no auto-degrade).
*   **AddToDeckTop has no animation**: invisible to players, may seem ineffective.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CopySetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **No i18n class key**: `RCG_CopySetting` is used as a description string ("Copy") but `AllTypes` shows stripped name "Copy"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | `[SerializeField] protected` |
| `m_CreateCardType` | CreateCardType | `CreateCardType` (enum) | — | Shared with `RCG_CardCreateSetting.CreateCardType` |

### A.3 Key methods
*   **`AddAction`**: select → per card `aCard.CopyCard()` → `CreateAction.CreateCard(copy, m_CreateCardType, InsertInOrder)`.
*   **`Infos`**: `RCG_CopySetting` + `CopySettingInfo` i18n.
*   **`GetDescriptionFormat`**: 3-way branch on `m_SelectHandCardSetting.m_SelectType`.

### A.4 Cross-system interactions
*   **`RCG_CardBattleData.CopyCard()`**: actual copy method.
*   **`CreateAction.CreateCard`**: insertion Action builder.
*   **`RCG_SelectHandCardSetting`**: selector trigger.
