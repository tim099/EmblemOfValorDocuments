---
title: Card Enhance Data (RCG_CardEnhenceData)
description: Card enhance "branch template" — enhance condition, add effect, change cost, change card type, ban use type
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card Enhance Data

> Class name: `RCG_CardEnhenceData`

## Purpose

**Template for a card enhance "branch"**. Each Asset describes "what an enhance does": add new effect, change cost, change card type, add tags, rename (e.g., α, β, γ). The card-side `RCG_CardData.m_EnhencePool` references an `RCG_CardEnhenceDropPool`; the pool drops these `RCG_CardEnhenceData` branches.

Inherits from `RCG_Asset<RCG_CardEnhenceData>`.

## Editor Layout

```
RCG_CardEnhenceData: <ID>
    LocalizeName              ← enhance name (α / β / γ; falls back to + when empty)
    Rarity                    ← enhance branch rarity
    Conditions                ← AND condition group (which cards can use this enhance)
    Effects                   ← new effects added by the enhance
    CardTags                  ← card tags added by the enhance
    EnhenceSettings           ← extra settings (modify existing effects via conditional set)
    BannedUsedType            ← block specific use types from this enhance
    CostAlter                 ← cost change (+/-)
    SetCardType / CardType    ← whether to force-change card type (e.g., Chant→Default)
```

## Main Fields

| Editor Display | Required | Description |
|---|---|---|
| **LocalizeName** | no | Enhance naming alternative (α / β / γ); falls back to `+` prefix when empty |
| **Rarity** | yes | Enhance branch rarity (affects drop weight) |
| **Conditions** | no | AND conditions: cards using this enhance must satisfy all |
| **Effects** | no | New effects added to the card |
| **CardTags** | no | Card tags added by the enhance (no duplication) |
| **EnhenceSettings** | no | Further modifications to existing effects (e.g., "OnPlay first effect, raise damage") |
| **BannedUsedType** | no | Disallowed use types (e.g., `Unplayable` cards forbidden from enhancing) |
| **CostAlter** | — | Cost delta (positive or negative) |
| **SetCardType** | — | Whether to force-change card type |
| **CardType** | when SetCardType=true | The new type (`Default` / `Chant`) |

## Behavior

### Condition Check (`CheckCondition`)
1. `m_BannedUsedType` contains target card's `UsedType` → not allowed.
2. `m_Conditions.CheckCondition(data)` fails → not allowed.
3. Each `EnhenceSettings.CheckCondition(data)` all pass → allowed.

### Application (`Enhence(card)`)
1. Write `m_LocalizeName` to the card's `m_EnhenceLocalize` (display).
2. Add all enabled `Effects` (deep clone).
3. `Cost += m_CostAlter`.
4. If `SetCardType`, apply new type.
5. `CardTags` → add tags (no duplication).
6. For each `EnhenceSettings`, run `Enhence(enhenceData)` (modify existing effects).

## Caveats

*   **Enhance is "permanent application"**: the cloned card is renamed / has new effects / cost changed; the original card is untouched.
*   **`BannedUsedType` usually includes `Unplayable`**: certain "non-playable" cards (system cards, hidden-effect cards) shouldn't be enhanced; the auto-add-`Unplayable` logic on deserialize was once present (now commented out).
*   **`EnhenceSettings` vs `Effects`**: `Effects` add new effects; `EnhenceSettings` modify existing ones (e.g., raise damage on the first damage effect).
*   **Empty `LocalizeName`** → uses default `+1` / `+2` display; non-empty replaces with `+α` / `α`.

---

## Appendix: Programmer Reference

### A.1 Class Info
*   **File**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_CardGames/RCG_CommonDatas/RCG_CardEnhenceData.cs`
*   **Inherits**: `RCG_Asset<RCG_CardEnhenceData>`
*   **AssetGroup**: `EditItems`

### A.2 Field Mapping

| Code Field | Editor Display | Type | Notes |
|---|---|---|---|
| `m_LocalizeName` | LocalizeName | `RCG_LocalizeData` | |
| `m_Rarity` | Rarity | `RCG_RarityTagGenData` | |
| `m_Conditions` | Conditions | `RCG_CE_AND_Condition` | AND group |
| `m_Effects` | Effects | `List<RCG_CommonEffect>` | |
| `m_CardTags` | CardTags | `List<RCG_CardTagGenData>` | |
| `m_EnhenceSettings` | EnhenceSettings | `List<RCG_CardEnhenceSetting>` | |
| `m_BannedUsedType` | BannedUsedType | `List<RCG_UsedTypeTagGenData>` | |
| `m_CostAlter` | CostAlter | `int` | |
| `m_SetCardType` | SetCardType | `bool` | |
| `m_CardType` | CardType | `CardType` enum | `Conditional(SetCardType)` |

### A.3 Key Methods

*   **`CheckCondition(EnhenceData)`** — three-layer check: BannedUsedType + Conditions + EnhenceSettings.
*   **`Enhence(RCG_CardData)`** — main application: write name → add effects → change cost → change cardType → add cardTags → apply EnhenceSettings.
*   **`EnhenceSettings` (property)** — filters `IsEnable = true` settings.

### A.4 System Interactions

*   **`RCG_CardData.m_EnhencePool`** / **`GetEnhenceBranchs`** — entry point for referencing this data.
*   **`RCG_CardEnhenceDropPool`** — random pool for enhance branches.
*   **`RCG_CardEnhenceCondition.EnhenceData`** — container passed during condition check / application.
*   **`RCG_CardEnhenceSetting`** — sub-setting for modifying existing effects.
*   **`RCG_CardEnhenceGenData`** — Asset Entry; default ID = `"Defense"`.

### A.5 Known Issues

*   `DeserializeFromJson`'s "auto-add `Unplayable` to `BannedUsedType`" logic is commented out, marking the legacy auto-behavior as deprecated.
