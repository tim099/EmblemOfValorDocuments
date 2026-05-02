---
title: Discard
description: Remove hand cards (discard / banish / top-deck / retain) with multiple selection modes and tag filters
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Discard

> Class: `RCG_CardDiscardSetting`

## Purpose
**Move hand cards out of hand** to the chosen destination (discard / banish / top-of-deck / retain). The core of "discard cards", "banish cards", and similar mechanics.

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **DiscardType** | Yes | Mode:<br>• **DiscardByCount** — player picks N<br>• **DiscardAll** — entire hand<br>• **DiscardAnyCount** — any number<br>• **RandomDiscardByCount** — random N<br>• **SelectedCards** — already chosen<br>• **SelectHandCard** — uses embedded selector |
| **DiscardCardNum** | When ByCount/RandomByCount | Cards to discard (variable). Conditional on type. |
| **RemoveType** | Yes | Destination: `Discard` / `Banish` / `ToDeckTop` / `TurnEndDiscard` (no discard effect) / `Retain`. |
| **SelectHandCardSetting** | When SelectHandCard | Embedded selector. |
| **DiscardCardTags** | No | Limit by card tag (e.g. "attack cards"). |
| **DiscardCardNotIncludedTags** | No | Inverse: only cards **without** these tags. |
| **DiscardVariable** | No | Stores actual discard count to a named variable for later use. |

## Behaviour
*   Per DiscardType: collects cards → `CreateAction.AddDiscardCardActions(...)` with the chosen `RemoveType`.
*   Stores count in `iData.VariableDic[DiscardVariable]` if set.
*   `DiscardAnyCount` shows `X` instead of `N` in description when `DiscardVariable` is set.
*   Description varies by `RemoveType` (e.g. `BanishCardIcon_Des` / `DiscardSelectedCardsIcon_Des`).

### Card fusion
Only `DiscardByCount` and `RandomDiscardByCount` modes fuse (counts add).

## Notes
*   **TurnEndDiscard skips discard effects**: designed for "lock in hand and force-clear at turn end"; not a generic discard.
*   **DiscardAnyCount has no DiscardCardNum**: use `DiscardByCount` for an upper bound; combine with `DiscardVariable` for "any".
*   **Variable name collisions**: multiple settings reusing `"X"` overwrite each other — name semantically.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardDiscardSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_CardDiscardSetting` → "Discard"
*   **Same-file nested**: `SelectCardSetting` (filter by `RCG_CardTagGenData` with `m_NotIncludedTags` and `m_WithoutEnhancement`)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_DiscardType` | DiscardType | enum | — | 6 modes |
| `m_DiscardCardNum` | DiscardCardNum | `IntVariable` | — | `[Conditional(... DiscardByCount, RandomDiscardByCount)]` |
| `m_RemoveType` | RemoveType | `RemoveType` (file enum) | — | 5 values |
| `m_SelectHandCardSetting` | SelectHandCardSetting | `RCG_SelectHandCardSetting` | — | `[Conditional(... SelectHandCard)]` |
| `m_DiscardCardTags` | DiscardCardTags | `List<RCG_CardTagGenData>` | — | |
| `m_DiscardCardNotIncludedTags` | DiscardCardNotIncludedTags | `bool` | — | |
| `m_DiscardVariable` | DiscardVariable | `string` | — | |

### A.3 Key methods
*   **`AddAction`**: 6-way branch on `m_DiscardType`; common path → `CreateAction.AddDiscardCardActions(iData, list, mode, RemoveType)`.
*   **`Fusion`**: only `DiscardByCount`/`RandomDiscardByCount`; clone + `IntVariable.FuseAdd`.
*   **`GetDescriptionFormat`**: branched i18n keys per `RemoveType` × `DiscardType` (e.g. `Banish{Type}Icon_Des`).

### A.4 Cross-system interactions
*   **`CreateAction.AddSelectCardAction / AddDiscardCardActions`**: UI selection + discard action builders.
*   **`RCG_GameManager.Random.RandomPick`**: random source.
*   **`SelectCardSetting`**: tag-filter helper (same file).
