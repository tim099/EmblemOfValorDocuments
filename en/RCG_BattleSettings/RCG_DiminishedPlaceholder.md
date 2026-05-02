---
title: Diminished placeholder
description: Auto-generated stand-in for diminished leaf effects; shows red "Diminished" text and resists further diminishment
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Diminished placeholder

> Class: `RCG_DiminishedPlaceholder`

## Purpose
**Replaces a leaf effect that was diminished** by `RCG_DiminishSetting`. The original effect (e.g. Attack) is swapped for this placeholder, displaying **red "Diminished"** text in descriptions.

> [!IMPORTANT]
> You typically **don't create this manually** — the diminish system auto-generates it at runtime. If it appears in your data, this card has been diminished before.

## Inspector layout
Generally absent from the new-setting dropdown (though listed in `AllTypes`); when present, shows no fields — it's purely a visual marker.

## Key fields
(none)

## Behaviour
*   No-op at trigger time (pure placeholder).
*   Description always shows "**Diminished**" in red (`RCG_Extensions.TagColors.Diminished`).
*   Treated as a leaf node — **cannot be further diminished** (empty fusion candidates).

## Notes
*   **Don't create manually**: unless intentionally pre-marking an effect as "diminished".
*   **vs Placeholder**: `RCG_PlaceholderSetting` is the **fusion system's** neutral placeholder (blue frame); this one is for the **diminish system** (red text).

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_DiminishedPlaceholder.cs`
*   **Inherits**: `RCG_PlaceholderSetting`
*   **No i18n class key**: editor shows stripped name `Diminished`

### A.2 Field map
(no fields of its own)

### A.3 Key methods
*   **`GetDescription / GetDescriptionShort / GetShortName`**: all return `UCL_LocalizeManager.Get("DiminishedPlaceholder").GetTagColor(TagColors.Diminished)`.
*   **`GetFusionCandidateSettings`** → empty list (not a fusion candidate).
*   **`GetFusionBaseSetting`** → `this` (preserves itself; **not further replaced**).

### A.4 Cross-system interactions
*   **`RCG_DiminishSetting.AddAction`**: triggers placeholder replacement.
*   **`RCG_CardBattleData.Diminish`**: replacement entry.
*   **`RCG_Extensions.TagColors.Diminished`**: red color source.
*   **i18n key `DiminishedPlaceholder`**: display text.
