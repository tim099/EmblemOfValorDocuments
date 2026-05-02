---
title: Card convert
description: Converts hand / selected / this card into a target card type
last_updated: 2026-05-02
target_audience: [Designer, Modder, AI_Agent]
---

# Card convert

> Class: `RCG_CardConvertSetting`

## Purpose
**Bulk-converts cards** to another type. Examples:
*   "Convert all hand cards into Undead cards"
*   "Convert this card to its enhanced form"
*   "Convert chosen cards to Trash"

Also used internally — when a unit dies, cards that can no longer be played auto-convert to "DeadCard".

## Key fields

| Inspector label | Required | Notes |
|---|---|---|
| **ConvertType** | Yes | Source:<br>• **SelectedCards** — cards already chosen by the player<br>• **AllHandCards** — entire hand<br>• **ThisCard** — the card triggering this effect |
| **ConvertTarget** | Yes | Target card template (`RCG_CardGenData`). |

## Behaviour
*   Description per ConvertType: i18n key `CardConvert_SelectedCards` / `CardConvert_AllHandCards` / `CardConvert_ThisCard`.
*   Runtime:
    1. Collect cards by ConvertType.
    2. `aDeck.Convert(old, new)` per card.
    3. **Hand cards** play a convert anim (0.2s interval each); cards in deck/discard swap silently.

## Notes
*   **ThisCard only valid in card-triggered context**: status-/event-triggered effects have no "this card" reference.
*   **SelectedCards needs prior selection**: typically preceded by `RCG_SelectHandCardSetting` to populate `iData.SelectedHandCards`.
*   **Invalid ConvertTarget ID**: NRE on `CardData.GetData()`. Verify the ID is registered.

---

## Appendix: Programmer Reference

### A.1 Class info
*   **Path**: `CardGame/Assets/Scripts/RCG_Scripts/RCG_GameDatas/RCG_BattleSettings/RCG_CardConvertSetting.cs`
*   **Inherits**: `RCG_BattleSetting`, `[System.Serializable]`
*   **i18n class key**: `RCG_CardConvertSetting` → "Card convert"

### A.2 Field map

| Code | Inspector | Type | Localize key | Notes |
|---|---|---|---|---|
| `m_ConvertType` | ConvertType | `ConvertType` (enum) | — | `SelectedCards` / `AllHandCards` / `ThisCard` |
| `m_ConvertTarget` | ConvertTarget | `RCG_CardGenData` | — | Target template |

### A.3 Key methods
*   **`AddAction`** (async):
    1. Collect `aCards` per `m_ConvertType` (hand / selected / this).
    2. Per card → clone new `RCG_CardBattleData` → `aDeck.Convert(old, new)`.
    3. Hand-position cards play `CardConvertAnim` (0.2s stagger); others swap silently.
*   **`RemoveDeadUnitCards(triggerEffectData)` (static)**: death hook — finds `!HaveRequireSkills(PlayerSkillTags)` cards and converts them to `RCG_CardData.DeadCardID`.
*   **`LocalizeKey`** → `"CardConvert_" + m_ConvertType.ToString()`.

### A.4 Cross-system interactions
*   **`RCG_Player.Ins.Deck.Convert`**: actual swap entry.
*   **`RCG_CardBattleData.CreateCard(target)`**: clone instance.
*   **`RCG_Card.CardConvertAnim`**: hand-card animation.
*   **`RCG_CardData.DeadCardID`**: target for death-cleanup conversions.
