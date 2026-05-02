---
title: Select hand card
description: Open a card chooser (or auto-pick) and write the result to SelectedHandCards for downstream settings
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Select hand card

> Class: `RCG_SelectHandCardSetting`

## Purpose
**Lets the player (or system) pick hand cards** — fills `SelectedHandCards` for downstream settings (Discard, Enhance, Copy, Fusion, CardToItem, ...). A "**preparatory setting**".

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **SelectType** | Yes | Selection mode (**13 options**, common ones below). |
| **SelectRange** | When SelectFromRange* | Range (Hand / Deck / DiscardPile, multi-select). |
| **SelectNum** | When ByCount/RandomSelect/RandomOneCardWithCost/LeftSide(Of)/RightSide | Count (or cost value). |
| **CardTags** | No | Filter by tag. |
| **SelectCardNotIncludedTags** | — | Inverse: cards **without** these tags. |
| **SelectCardWithoutEnhancement** | — | Restrict to non-enhanced cards. |
| **SelectCountVariable** | No | Save the count to a named variable. |
| **DetailSetting** | No | Description tweaks (`Default` / `Then` / `SelectPrefix`). |

### Common SelectType values
| Value | Behaviour |
|---|---|
| **SelectByCount** | Player picks N (most common, default) |
| **SelectAnyCount** | Player picks any number |
| **SelectAll** | Auto-select all (no UI) |
| **SkipSelect** | Skip — already chosen earlier |
| **CreatedCards** | Auto-select cards created in this trigger |
| **RandomSelect** | Random pick of N |
| **SelectFromRange** | Pick from Hand / Deck / Discard mix |
| **SelectFromRangeAll** | Auto-select all in given range |
| **ThisCard** / **TriggeredCard** | The triggering card |
| **LeftSide** / **RightSide** | N cards to the left / right |
| **LeftSideOfThis** | Left of this card |
| **RandomOneCardWithCost** | Random card with cost = X (X via `SelectNum`) |

## Behaviour
*   **Runtime**: opens chooser UI (or auto-picks per mode), writes result to `iData.SelectedHandCards`.
*   **Downstream consumption**: subsequent settings (Discard, Enhance, ...) read `iData.SelectedHandCards`.
*   **Variable write**: with `SelectCountVariable`, writes selected count for downstream use.
*   **Description**: branches on `DetailSetting.DescriptionType`.

## Notes
*   **Rarely used standalone**: this only **selects**; downstream settings act on the result. **Wire something after** Discard / Enhance / Copy / Fusion / etc.
*   **AI usage**: chooser UI is player-only; AI should pick `RandomSelect` / `SelectAll` / `SkipSelect` / etc.
*   **SelectFromRange empty list**: equivalent to "no selection".

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_SelectHandCardSetting.cs`
*   **Inherits**: `RCG_BattleSetting`
*   **i18n class key**: `RCG_SelectHandCardSetting` → "Select hand card"
*   **Same-file types**: `SelectType` (13 values), `DescriptionType` (3 values), nested `DetailSetting` (with `m_DescriptionType`)

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_SelectType` | SelectType | enum | — | 13 modes |
| `m_SelectRange` | SelectRange | `List<CardPos>` | — | `[Conditional(nameof(m_SelectType), false, SelectFromRange, SelectFromRangeAll)]` |
| `m_SelectNum` | SelectNum | `IntVariable` | — | `[Conditional(... 6 modes that need a count)]` |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | — | |
| `m_SelectCardNotIncludedTags` | SelectCardNotIncludedTags | `bool` | — | |
| `m_SelectCardWithoutEnhancement` | SelectCardWithoutEnhancement | `bool` | — | |
| `m_SelectCountVariable` | SelectCountVariable | `string` | — | |
| `m_DetailSetting` | DetailSetting | `DetailSetting` (file nested) | `DetailSetting` | |

### A.3 Key methods
*   **`AddAction`** (beyond shown range): 14-way branch on `m_SelectType` — UI / auto / range / random; final step writes `iData.SelectedHandCards`.
*   **`TagDes` (private)**: tag description; includes `NotIncludedDes` and `SelectCardWithoutEnhancementDes` modifiers.
*   **`GetDescriptionFormat / Params`**: shape varies per `m_DetailSetting.m_DescriptionType`.

### A.4 Cross-system interactions
*   **`iData.SelectedHandCards`**: write target; downstream `IntVariable` resolution reads it.
*   **`CardPos` enum**: range options (Hand / Deck / DiscardPile).
*   **`CreateAction.AddSelectCardAction`**: chooser UI entry.
